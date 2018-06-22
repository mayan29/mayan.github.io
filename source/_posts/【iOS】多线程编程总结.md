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











### 死锁问题

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

### 任务组

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


## 参考资料

1. [一篇专题让你秒懂GCD死锁问题!](https://www.jianshu.com/p/201ccb40a3f8)
2. [iOS 多线程编程总结](https://bestswifter.com/multithreadconclusion/#)








