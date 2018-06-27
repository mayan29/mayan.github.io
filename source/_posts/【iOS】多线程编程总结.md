---
title: 【iOS】多线程编程总结
date: 2016-12-22 12:56:31
categories: iOS
tags:
---


多线程在 iOS 开发中有以下四类：

| 多线程 | 特点 | 线程管理 | 使用几率 |
| --- | --- | --- | --- |
| pthread | 跨平台、使用难度大 | 程序员管理线程 | 几乎不用 |
| NSThread | 更加面向对象、简单易用、直接操作线程对象 | 程序员管理线程 | 偶尔使用 |
| GCD | 代替 NSThread、充分利用设备内核 | 自动管理线程 | 经常使用 |
| NSOperation | 基于 GCD、更加面向对象 | 自动管理线程 | 经常使用 |

多线程的优点：

- 能适当提高程序的执行效率
- 能适当提高资源利用率（CPU、内存利用率）

多线程的缺点：

- 开启线程需要占用一定的内存空间（默认情况下，主线程占用 1M，子线程占用 512KB），如果开启大量的线程，会占用大量的内存空间，降低程序的性能
- 线程越多，CPU 在调度线程上的开销就越大
- 程序设计更加复杂：比如线程之间的通信、多线程的数据共享


## GCD

GCD（Grand Central Dispatch，牛逼的中枢调度器）是 iOS 4.0 引入的多线程编程技术，其有两个核心的概念：任务和队列。GCD 会自动将队列中的任务取出来，放到对应的线程中执行。

> 线程，是执行程序最基本的单元，它有自己栈和寄存器。说得再具体一些，线程就是一个 CPU 执行的一条无分叉的命令列。对于多线程，其中每一条线程都会有自己的栈和寄存器。

### 1. 同步 / 异步 & 串行队列 / 并行队列

> 同步（sync） ：会阻塞当前线程并等待 block 中的任务执行完毕，然后当前线程才会继续往下运行。
> 异步（async）：当前线程会直接往下进行，不会阻塞当前线程。
> 串行队列（Serial Dispatch Queue）：任务顺次执行，取出来一个，执行一个，然后再取下一个，这样一个一个执行。
> 并发队列（Serial Dispatch Queue）：任务并发执行，取出来一个，放到另外的线程执行，然后再取下一个，这样看起来都是一起执行的。但是 GCD 会根据系统资源控制并行的数量，所以如果任务很多，并不会让所有任务同时执行。

```objc
// 1. 同步执行 & 串行队列 —— 当前线程，一个一个执行，没有意义，略

// 2. 同步执行 & 并行队列 —— 当前线程，一个一个执行，同样没有意义，略
    
// 3. 异步执行 & 串行队列 —— 其他线程，一个一个执行
    
dispatch_queue_t queue = dispatch_queue_create("com.mayan29.queue", DISPATCH_QUEUE_SERIAL);
dispatch_async(queue, ^{
    NSLog(@"执行任务 A, 线程 %@", [NSThread currentThread]);
});
dispatch_async(queue, ^{
    NSLog(@"执行任务 B, 线程 %@", [NSThread currentThread]);
});
dispatch_async(queue, ^{
    NSLog(@"执行任务 C, 线程 %@", [NSThread currentThread]);
});
    
// 打印结果：
// 执行任务 A, 线程 <NSThread: 0x60000026bcc0>{number = 3, name = (null)}
// 执行任务 B, 线程 <NSThread: 0x60000026bcc0>{number = 3, name = (null)}
// 执行任务 C, 线程 <NSThread: 0x60000026bcc0>{number = 3, name = (null)}
    
// 4. 异步执行 & 并行队列 —— 新开启多条线程，一起执行
    
dispatch_queue_t queue = dispatch_queue_create("com.mayan29.queue", DISPATCH_QUEUE_CONCURRENT);
dispatch_async(queue, ^{
    NSLog(@"执行任务 A, 线程 %@", [NSThread currentThread]);
});
dispatch_async(queue, ^{
    NSLog(@"执行任务 B, 线程 %@", [NSThread currentThread]);
});
dispatch_async(queue, ^{
    NSLog(@"执行任务 C, 线程 %@", [NSThread currentThread]);
});
    
// 打印结果：
// 执行任务 B, 线程 <NSThread: 0x6000002783c0>{number = 4, name = (null)}
// 执行任务 C, 线程 <NSThread: 0x60400027a6c0>{number = 5, name = (null)}
// 执行任务 A, 线程 <NSThread: 0x600000277800>{number = 3, name = (null)}
```

### 2. 死锁问题

在使用 GCD 的过程中，如果向当前串行队列中同步派发一个任务，就会导致死锁。比如下面两段代码都会导致死锁：

```objc
dispatch_sync(dispatch_get_main_queue(), ^{
    NSLog(@"任务 A");
});    
NSLog(@"任务 B");
```

```objc
dispatch_queue_t queue = dispatch_queue_create("com.mayan29.queue", NULL);
dispatch_async(queue, ^{
    dispatch_sync(queue, ^{
        NSLog(@"任务 A");
    });
    NSLog(@"任务 B");
});
```

程序遇到了同步线程，那么它会进入等待，等待任务 A 执行完，然后执行任务 B。但对于串行队列，有任务来就会将任务加到队尾，然后遵循 FIFO 原则执行任务。那么，现在任务 A 就会被加到最后，任务 B 排在了任务 A 前面，那么问题来了：

任务 B 要等任务 A 执行完才能执行，任务 A 又排在任务 B 后面，意味着任务 A 要在任务 B 执行完才能执行，所以他们进入了互相等待的局面，这就是死锁。

其实在通常情况下我们不必要用 dispatch_sync，因为 dispatch_async 能够更好的利用 CPU，提升程序运行速度。只有当我们需要保证队列中的任务必须顺序执行时，才考虑使用 dispatch_sync。在使用 dispatch_sync 的时候应该分析当前处于哪个队列，以及任务会提交到哪个队列。

### 3. 变更优先级

```objc
dispatch_queue_t queue = dispatch_queue_create("com.mayan29.queue", DISPATCH_QUEUE_CONCURRENT);
dispatch_set_target_queue(queue, dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_LOW, 0));
```    

### 4. 任务组

```objc
dispatch_group_t group = dispatch_group_create();
dispatch_queue_t queue = dispatch_queue_create("com.mayan29.queue", NULL);

dispatch_group_async(group, queue, ^{
    NSLog(@"执行 A 任务");
});
dispatch_group_async(group, queue, ^{
    NSLog(@"执行 B 任务");
});
dispatch_group_async(group, queue, ^{
    NSLog(@"执行 C 任务");
});

dispatch_group_notify(group, queue, ^{
    NSLog(@"任务全部执行完毕");
});
```

```objc
dispatch_group_t group = dispatch_group_create();
dispatch_queue_t queue = dispatch_queue_create("com.mayan29.queue", NULL);
    
dispatch_group_async(group, queue, ^{
    NSLog(@"执行 A 任务");
});
dispatch_group_async(group, queue, ^{
    NSLog(@"执行 B 任务");
});
dispatch_group_async(group, queue, ^{
    sleep(5);
    NSLog(@"执行 C 任务");
});
    
// 等待 2 秒，如果是不限时间可改为 DISPATCH_TIME_FOREVER
long result = dispatch_group_wait(group, dispatch_time(DISPATCH_TIME_NOW, 2 * NSEC_PER_SEC));
if (result == 0) {
    NSLog(@"任务全部执行完成");
} else {
    NSLog(@"任务正在处理");
}
    
dispatch_group_notify(group, queue, ^{
    NSLog(@"任务全部执行完毕");
});
```

打印结果
    
```objc
2016-12-21 14:53:39.334517+0800 GCD[13489:5363201] 执行 A 任务
2016-12-21 14:53:39.334677+0800 GCD[13489:5363201] 执行 B 任务
2016-12-21 14:53:44.335772+0800 GCD[13489:5363152] 任务正在处理
2016-12-21 14:53:47.340231+0800 GCD[13489:5363201] 执行 C 任务
2016-12-21 14:53:47.340474+0800 GCD[13489:5363201] 任务全部执行完毕
```

### 5. 栅栏函数

```objc
dispatch_queue_t queue = dispatch_queue_create("com.mayan29.queue", NULL);

dispatch_async(queue, ^{
    NSLog(@"执行 A 任务");
});
dispatch_async(queue, ^{
    NSLog(@"执行 B 任务");
});
    
dispatch_barrier_async(queue, ^{
    NSLog(@"任务 A 和 B 执行完毕，即将执行任务 C");
});
    
dispatch_async(queue, ^{
    NSLog(@"执行 C 任务");
});
```

### 6. Apply 循环执行

```objc
NSArray *arr = @[@"执行 A 任务", @"执行 B 任务", @"执行 C 任务"];

dispatch_queue_t queue = dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0);
dispatch_async(queue, ^{
    dispatch_apply(arr.count, queue, ^(size_t index) {
        NSLog(@"%@", arr[index]);
    });
    NSLog(@"任务全部执行完毕");
});
```

### 7. 队列唤醒 & 挂起

```objc
dispatch_queue_t queue = dispatch_queue_create("com.mayan29.queue", NULL);
    
dispatch_async(queue, ^{
    NSLog(@"执行 A 任务");
});
    
dispatch_suspend(queue);  // 挂起
```

```objc
dispatch_resume(queue);  // 唤醒，执行任务 A
```

### 8. 信号量

其实最经典的例子就是批量下载。比如有 N 个图片需要一个一个下载，但是需要控制每次只能下载 3 个，就需要用到信号量了。

```objc
dispatch_semaphore_t semaphore = dispatch_semaphore_create(3);
    
dispatch_group_t group = dispatch_group_create();

dispatch_group_async(group, dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT,0), ^{
    dispatch_semaphore_wait(semaphore, DISPATCH_TIME_FOREVER);
    NSLog(@"下载图片 A");
    sleep(2);
    dispatch_semaphore_signal(semaphore);
});
dispatch_group_async(group, dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT,0), ^{
    dispatch_semaphore_wait(semaphore, DISPATCH_TIME_FOREVER);
    NSLog(@"下载图片 B");
    sleep(2);
    dispatch_semaphore_signal(semaphore);
});
dispatch_group_async(group, dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT,0), ^{
    dispatch_semaphore_wait(semaphore, DISPATCH_TIME_FOREVER);
    NSLog(@"下载图片 C");
    sleep(2);
    dispatch_semaphore_signal(semaphore);
});
dispatch_group_async(group, dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT,0), ^{
    dispatch_semaphore_wait(semaphore, DISPATCH_TIME_FOREVER);
    NSLog(@"下载图片 D");
    sleep(2);
    dispatch_semaphore_signal(semaphore);
});
dispatch_group_async(group, dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT,0), ^{
    dispatch_semaphore_wait(semaphore, DISPATCH_TIME_FOREVER);
    NSLog(@"下载图片 E");
    sleep(2);
    dispatch_semaphore_signal(semaphore);
 });

dispatch_group_notify(group, dispatch_get_main_queue(), ^{
    NSLog(@"任务全部执行完毕");
});
```  

打印结果

```objc
2016-12-21 16:56:44.447412+0800 GCD[15452:5524785] 下载图片 C
2016-12-21 16:56:44.447412+0800 GCD[15452:5524781] 下载图片 A
2016-12-21 16:56:44.447412+0800 GCD[15452:5524782] 下载图片 B
2016-12-21 16:56:46.449871+0800 GCD[15452:5524784] 下载图片 D
2016-12-21 16:56:46.449866+0800 GCD[15452:5524783] 下载图片 E
2016-12-21 16:56:48.455010+0800 GCD[15452:5524743] 任务全部执行完毕
```

### 9. 定时器

GCD 定时器不受 RunLoop 中 Mode 的影响（RunLoop 内部也是基于 GCD 实现的)，比如滚动 TableView 的时候，GCD 的定时器不受影响。

```objc
@property (nonatomic, strong) dispatch_source_t timer;



self.timer = dispatch_source_create(DISPATCH_SOURCE_TYPE_TIMER, 0, 0, dispatch_get_global_queue(0, 0));
    
dispatch_time_t start = dispatch_time(DISPATCH_TIME_NOW, 3 * NSEC_PER_SEC);  // 3 秒以后开始
dispatch_source_set_timer(self.timer, start, 1 * NSEC_PER_SEC, 0);  // 间隔 1 秒
dispatch_source_set_event_handler(self.timer, ^{
       
    NSLog(@"执行");
        
    // 10 秒之后停止
    dispatch_after(dispatch_time(DISPATCH_TIME_NOW, (int64_t)(10 * NSEC_PER_SEC)), dispatch_get_main_queue(), ^{
        dispatch_cancel(self.timer);
    });
});
    
// 启动定时器
dispatch_resume(self.timer);
```


## NSOperation

从简单意义上来说，NSOperation 就是对 GCD 中的 block 进行的封装。相对 GCD 来说，使用 NSOperation 会增加一点点额外的开销，但是我们却换来了非常强大的灵活性和功能，我们可以给 operation 之间添加依赖关系、取消一个正在执行的 operation 、暂停和恢复 operation queue 等。并且它有三种状态 isExecuted、isFinished 和 isCancelled 以方便我们通过 KVC 对它的状态进行监听。

### 1. 异步并发队列

用 NSBlockOperation 是因为 NSOperation 是一个基类，不应该直接生成 NSOperation 对象，而是应该用它的子类。NSBlockOperation 是苹果预定义的子类，它可以用来封装一个或多个 block。

#### 标准创建

```objc
// 1. 创建队列
NSOperationQueue *queue = [[NSOperationQueue alloc] init];

// 2. 创建操作
NSBlockOperation *operation = [NSBlockOperation blockOperationWithBlock:^{
    NSLog(@"执行任务 A, 线程 %@", [NSThread currentThread]);
}];
[operation addExecutionBlock:^{
    NSLog(@"执行任务 B, 线程 %@", [NSThread currentThread]);
}];
[operation addExecutionBlock:^{
    NSLog(@"执行任务 C, 线程 %@", [NSThread currentThread]);
}];

// 3. 添加操作到队列，自动异步执行
[queue addOperation:operation];
    
// 打印结果：
// 执行任务 B, 线程 <NSThread: 0x6000002783c0>{number = 4, name = (null)}
// 执行任务 A, 线程 <NSThread: 0x600000277800>{number = 3, name = (null)}
// 执行任务 C, 线程 <NSThread: 0x60400027a6c0>{number = 5, name = (null)}
```

#### 快速创建

```objc
NSOperationQueue *queue = [[NSOperationQueue alloc] init];
    
[queue addOperationWithBlock:^{
    NSLog(@"执行任务 A, 线程 %@", [NSThread currentThread]);
}];
[queue addOperationWithBlock:^{
    NSLog(@"执行任务 B, 线程 %@", [NSThread currentThread]);
}];
[queue addOperationWithBlock:^{
    NSLog(@"执行任务 C, 线程 %@", [NSThread currentThread]);
}];
    
// 打印结果：
// 执行任务 B, 线程 <NSThread: 0x60400026eb80>{number = 3, name = (null)}
// 执行任务 C, 线程 <NSThread: 0x600000273700>{number = 5, name = (null)}
// 执行任务 A, 线程 <NSThread: 0x60400007c500>{number = 4, name = (null)}
```

#### 调用方式创建

```objc
NSOperationQueue *queue = [[NSOperationQueue alloc] init];
    
NSInvocationOperation *operationA = [[NSInvocationOperation alloc] initWithTarget:self selector:@selector(methodA) object:nil];
NSInvocationOperation *operationB = [[NSInvocationOperation alloc] initWithTarget:self selector:@selector(methodB) object:nil];
NSInvocationOperation *operationC = [[NSInvocationOperation alloc] initWithTarget:self selector:@selector(methodC) object:nil];
    
[queue addOperation:operationA];
[queue addOperation:operationB];
[queue addOperation:operationC];

- (void)methodA {
    NSLog(@"执行任务 A, 线程 %@", [NSThread currentThread]);
}
- (void)methodB {
    NSLog(@"执行任务 B, 线程 %@", [NSThread currentThread]);
}
- (void)methodC {
    NSLog(@"执行任务 C, 线程 %@", [NSThread currentThread]);
}
```

### 2. 最大并发数 & 依赖关系

用 NSOperation 模仿一下 GCD 的信号量 & 任务组。可能是我没有理解到位？反正我还是感觉 GCD 最好用，清晰明了。

```objc
// 1. 创建队列
NSOperationQueue *queue = [[NSOperationQueue alloc] init];
    
// 2. 限制每次最多并发 3 个
queue.maxConcurrentOperationCount = 3;
    
// 2. 创建操作
NSBlockOperation *operationA = [NSBlockOperation blockOperationWithBlock:^{
    NSLog(@"下载图片 A");
    sleep(2);
}];
NSBlockOperation *operationB = [NSBlockOperation blockOperationWithBlock:^{
    NSLog(@"下载图片 B");
    sleep(2);
}];
NSBlockOperation *operationC = [NSBlockOperation blockOperationWithBlock:^{
    NSLog(@"下载图片 C");
    sleep(2);
}];
NSBlockOperation *operationD = [NSBlockOperation blockOperationWithBlock:^{
    NSLog(@"下载图片 D");
    sleep(2);
}];
NSBlockOperation *operationE = [NSBlockOperation blockOperationWithBlock:^{
    NSLog(@"下载图片 E");
    sleep(2);
}];
NSBlockOperation *operationEnd = [NSBlockOperation blockOperationWithBlock:^{
    NSLog(@"任务全部执行完毕");
}];
    
// 3. 添加操作到队列，自动异步执行
[queue addOperation:operationA];
[queue addOperation:operationB];
[queue addOperation:operationC];
[queue addOperation:operationD];
[queue addOperation:operationE];
[queue addOperation:operationEnd];
    
// 4. 添加所有依赖
[operationEnd addDependency:operationA];
[operationEnd addDependency:operationB];
[operationEnd addDependency:operationC];
[operationEnd addDependency:operationD];
[operationEnd addDependency:operationE];
```

打印结果

```objc
2016-12-22 14:33:16.295008+0800 GCD[35716:6312067] 下载图片 A
2016-12-22 14:33:16.295008+0800 GCD[35716:6312068] 下载图片 C
2016-12-22 14:33:16.295008+0800 GCD[35716:6312069] 下载图片 B
2016-12-22 14:33:18.298938+0800 GCD[35716:6312068] 下载图片 E
2016-12-22 14:33:18.298903+0800 GCD[35716:6312070] 下载图片 D
2016-12-22 14:33:20.301242+0800 GCD[35716:6312070] 任务全部执行完毕
```

### 3. 取消任务

如果我们有两次网络请求，第二次请求会用到第一次的数据。如果此时网络情况不好，第一次请求超时了，那么第二次请求也没有必要发送了。当然，用户也有可能人为地取消某个 NSOperation。

当某个 NSOperation 被取消时，我们应该尽可能的清除 NSOperation 内部的数据并且把 cancelled 和 finished 设为 true，把 executing 设为 false。

```objc
[operation cancel]; // 取消某个 operation
```

### 4. 其他方法

```objc
NSOperationQueue *queue = [NSOperationQueue mainQueue];  // 主队列
[queue cancelAllOperations];  // 取消队列所有操作
[queue setSuspended:YES];  // YES暂停，NO恢复队列
``` 


## NSThread

这套方案是经过苹果封装后的，并且完全面向对象的，所以可以直接操控线程对象，非常直观和方便。但是，它的生命周期还是需要我们手动管理，所以这套方案也是偶尔用用，比如 [NSThread currentThread]。

### 创建多线程

```objc
// 标准创建
NSThread *thread = [[NSThread alloc] initWithTarget:self selector:@selector(download) object:nil];
[thread start];

// 快速创建
[NSThread detachNewThreadSelector:@selector(download) toTarget:self withObject:nil];

// 隐式创建
[self performSelectorInBackground:@selector(download) withObject:nil];
```

### 其他方法

```objc
// 判断某个线程的状态的属性
@property (readonly, getter=isExecuting) BOOL executing;
@property (readonly, getter=isFinished)  BOOL finished;
@property (readonly, getter=isCancelled) BOOL cancelled;

// 获取当前线程 / 获取主线程
@property (class, readonly, strong) NSThread *currentThread;
@property (class, readonly, strong) NSThread *mainThread;
// 当前线程是否是主线程
@property (class, readonly) BOOL isMainThread;

// 取消线程
- (void)cancel;

// 使当前线程暂停 N 秒
+ (void)sleepUntilDate:(NSDate *)date;
+ (void)sleepForTimeInterval:(NSTimeInterval)ti;

// 强制关闭线程
+ (void)exit;
```

### 线程加锁

```objc
- (void)touchesBegan:(NSSet<UITouch *> *)touches withEvent:(UIEvent *)event {
    
    _ticketNum = 100;
    
    _thread_1 = [[NSThread alloc] initWithTarget:self selector:@selector(saleTicket) object:nil];
    _thread_2 = [[NSThread alloc] initWithTarget:self selector:@selector(saleTicket) object:nil];
    _thread_3 = [[NSThread alloc] initWithTarget:self selector:@selector(saleTicket) object:nil];
    
    _thread_1.name = @"一号窗口";
    _thread_2.name = @"二号窗口";
    _thread_3.name = @"三号窗口";
    
    [_thread_1 start];
    [_thread_2 start];
    [_thread_3 start];
}

- (void)saleTicket {
    
    while (1) {
        @synchronized (self) {  // 加锁
            
            if (_ticketNum > 0) {
                
                _ticketNum --;
                NSLog(@"%@卖了一张票，剩余%d张票", [NSThread currentThread].name, _ticketNum);
                
            } else {
                return;  // 退出循环
            }
            
        }  // 解锁
    }
}
```


## 参考资料

1. [一篇专题让你秒懂GCD死锁问题!](https://www.jianshu.com/p/201ccb40a3f8)
2. [iOS 多线程编程总结](https://bestswifter.com/multithreadconclusion/#)
3. [Objective-C 的底层并发 API](http://www.cocoachina.com/industry/20130821/6842.html)
4. [iOS 开发系列 -- 并行开发其实很容易](http://www.cnblogs.com/kenshincui/p/3983982.html)
5. [关于 iOS 多线程，你看我就够了](https://www.jianshu.com/p/0b0d9b1f1f19)