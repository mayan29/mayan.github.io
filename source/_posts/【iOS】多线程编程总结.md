---
title: 【iOS】多线程编程总结
date: 2016-12-22 12:56:31
categories: iOS
tags:
---


## GCD

GCD（Grand Central Dispatch，牛逼的中枢调度器）是 iOS 4.0 引入的多线程编程技术，其有两个核心的概念：任务和队列。GCD 会自动将队列中的任务取出来，放到对应的线程中执行。

> 线程，是执行程序最基本的单元，它有自己栈和寄存器。说得再具体一些，线程就是一个 CPU 执行的一条无分叉的命令列。对于多线程，其中每一条线程都会有自己的栈和寄存器。

### 1. 同步 / 异步 & 串行队列 / 并发队列

> 同步（sync） ：只能在当前线程中执行任务
> 异步（async）：可以在新的线程中执行任务
> 串行队列（Serial Dispatch Queue）：任务顺次执行
> 并发队列（Serial Dispatch Queue）：任务并发执行

```objc
// 1. 同步执行 & 串行队列 —— 在当前线程顺次执行任务，没有意义，略

// 2. 同步执行 & 并行队列 —— 在当前线程并发执行，同样没有意义，略
    
// 3. 异步执行 & 串行队列 —— 新开启一条线程顺次执行任务
    
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
    
// 4. 异步执行 & 并行队列 —— 新开启多条线程并发执行任务
    
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


## 参考资料

1. [一篇专题让你秒懂GCD死锁问题!](https://www.jianshu.com/p/201ccb40a3f8)
2. [iOS 多线程编程总结](https://bestswifter.com/multithreadconclusion/#)
3. [Objective-C 的底层并发 API](http://www.cocoachina.com/industry/20130821/6842.html)








