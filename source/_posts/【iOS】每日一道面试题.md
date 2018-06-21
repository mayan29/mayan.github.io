---
title: 【iOS】每日一道面试题
date: 2018-06-19 10:21:55
categories: iOS
tags:
---

坚持每日记录一道面试题，持续更新。★ 为超级简单题目，★★ 为简单题目，以此类推，满级 ★★★★★ 五星。



# C / Objective-C


## 1

__题目：把 "www.zhidao.baidu.com" 字符串改成 "com/baidu/zhidao/www"。__ 
__星级：★__  

```objc
NSString *str = @"www.zhidao.baidu.com";
NSArray *tmpArray = [str componentsSeparatedByString:@"."];
tmpArray = [[tmpArray reverseObjectEnumerator] allObjects];  // 数组倒叙排列
NSString *resultStr = [tmpArray componentsJoinedByString:@"/"];
NSLog(@"%@", resultStr);  
```

打印结果

```objc
com/baidu/zhidao/www
```

> 其实这道题一点难点都没有，唯一需要注意的是要熟记这三个方法的方法名。
> 
> components - n. 组件
> separated - adj. 分离的
> reverse - v. 反转，颠倒
> enumerator - n. 枚举器


## 2

__题目：不用临时变量怎么实现 swap(a, b) ？__
__星级：★__  

```objc
int a = 10;
int b = 20;
    
NSLog(@"a = %d, b = %d", a, b);  

a = a ^ b;
b = a ^ b;
a = a ^ b;
    
NSLog(@"a = %d, b = %d", a, b);  
```

打印结果

```objc
a = 10, b = 20
a = 20, b = 10
```

> 用加法或者异或都可以，我比较偏好于异或，因为如果使用加法，两个数值太大的情况下，容易超出 int 范围。


## 3

__题目：下面这段代码的输出结果是？__
__星级：★★__

```c
int a[5] = {1, 2, 3, 4, 5};
int *ptr = (int *)(&a + 1);
    
printf("%d, %d", *(a + 1), *(ptr - 1));
```

打印结果

```objc
2, 5
```

a 是指向数组开头元素的指针，a + 1 就是指向下一个元素的指针，所以星号求值以后是 2。&a 相当于是数组的指针，&a + 1 是数组后面一个数组的指针，然后转换成 int * 类型是 5 这个数字后面的一个数字的指针。再减一就是指向 5 的指针，所以星号求值以后是 5。



# Foundation / UIKit


## 1

__题目：OC 中 load 和 initialize 方法有何异同？__
__星级：★★__

相同点：

1. load 和 initialize 方法都会在实例化对象之前调用，main 函数之前调用 load，之后调用 initialize；
2. load 和 initialize 方法都只会调用一次；
3. load 和 initialize 方法都会被自动调用，不能手动调用它们；
4. load 和 initialize 方法调用，会从父到子依次实现父类们的 load 和 initialize 方法；
5. load 和 initialize 方法内部为了保证线程安全都使用了锁，实现时要保持逻辑简单，避免阻塞线程；

不同点：

1. load 方法通常用来进行 Method Swizzle，initialize 方法通常用于初始化全局变量或者静态变量。
2. load 方法在类被程序装载时调用，与这个类是否被用到无关，initialize 方法是在类或者它的子类接收第一条消息前被调用。如果该类一直没有被用到，它的 initialize 方法也不会调用。

其他：

1. load 方法调用顺序永远都是：parent > child > child+extend ；
2. 当有 child+extend 存在的时候，child+extend 中的 initialize 方法会覆盖 child 中的 initialize 方法。

具体分析和实例参考：[细说 OC 中的 load 和 initialize 方法](https://bestswifter.com/load-and-initialize/)



# 待总结题目

1. 说说你对 block 的理解。 —— 三种 block，栈上的 block 在 ARC 下会自动复制到堆上，block 的属性修饰符是 copy，循环引用的原理和解决方案。
2. block 为什么会有循环引用
2. 说说你对 runtime 的理解。 —— 主要是方法调用时如何查找缓存，如何找到方法，找不到方法时怎么转发，对象的内存布局。
3. 说说你对 MVC、MVVM 和 RAC 的理解。 —— MVC 的 C 太臃肿，可以和 V 合并，变成 MVVM 中的 V，而 VM 用来将 M 转化成 V 能用的数据。RAC 数据的双向绑定怎么做，bind 函数了解过么。
4. MVC 具有什么样的优势，各个模块之间怎么通信，比如点击 Button 后 怎么通知 Model？
5. 说说 UITableView 的调优。 —— 一方面是通过 instruments 检查影响性能的地方，另一方面是估算高度并在 runloop 空闲时缓存。
6. 谈谈你对 ARC 的理解。 —— ARC 是编译器完成的，依靠引用计数，谈谈几个属性修饰符的内存管理策略，什么情况下会内存泄露。
7. 野指针是什么，iOS 开发中什么情况下会有野指针？ —— 野指针是不为 nil，但是指向已经被释放的内存的指针。
8. 介绍自己用过哪些开源库。 —— Masonry 和 SnapKit，AFNetWorking，MKNetworkKit，Alamofire，Mantle，SDWebImage
9. SDWebImage 下载了图片后为什么要解码？ —— 要把 png 文件建立一个内存映射。
10. 两个无限长度链表（也就是可能有环） 判断有没有交点
11. KVO、Notification、delegate 各自的优缺点，效率还有使用场景
12. 如何手动通知 KVO
13. Objective-C 中的 copy 方法
14. runtime 中，SEL 和 IMP 的区别
15. autoreleasepool 的使用场景和原理
16. RunLoop 的实现原理和数据结构，什么时候会用到
17. 使用 GCD 如何实现这个需求：A、B、C 三个任务并发，完成后执行任务 D。
18. NSOperation 和 GCD 的区别
19. CoreData 的使用，如何处理多线程问题
20. 如何设计图片缓存？
21. 有没有自己设计过网络控件？
22. 怎么判断某个 cell 是否显示在屏幕上
23. 进程和线程的区别
24. TCP 与 UDP 区别
24. TCP 流量控制
24. 数组和链表的区别
24. UIView 生命周期
24. 如果页面 A 跳转到 页面 B，A 的 viewDidDisappear 方法和 B 的 viewDidAppear 方法哪个先调用？
24. ARC 的本质
24. RunLoop 的基本概念，它是怎么休眠的？
24. Autoreleasepool 什么时候释放，在什么场景下使用？
24. 如何找到字符串中第一个不重复的字符
24. 哈希表如何处理冲突



# 参考资料

1. [让 BAT 的 Offer 不再难拿 - bestswifter](https://bestswifter.com/bat-interview/)