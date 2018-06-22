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


## 4

__题目：使用 GCD 如何实现这个需求：A、B、C 三个任务并发，完成后执行任务 D。__
__星级：★★__

可以用两种方法实现：

```objc
// GCD Group

dispatch_group_t group = dispatch_group_create();
dispatch_queue_t queue = dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0);

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
    NSLog(@"执行 D 任务");
});
```

```objc
// GCD Barrier

dispatch_queue_t queue = dispatch_queue_create("com.mayan29.gcd", DISPATCH_QUEUE_CONCURRENT);

dispatch_async(queue, ^{
    NSLog(@"执行 A 任务");
});
dispatch_async(queue, ^{
    NSLog(@"执行 B 任务");
});
dispatch_async(queue, ^{
    NSLog(@"执行 C 任务");
});
    
dispatch_barrier_async(queue, ^{
    NSLog(@"可以做一些其他操作");
});
    
dispatch_async(queue, ^{
    NSLog(@"执行 D 任务");
});
```    



# UIKit



# 操作系统


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


## 2

__题目：如果页面 A 跳转到 页面 B，A 的 viewDidDisappear 方法和 B 的 viewDidAppear 方法哪个先调用？__
__星级：★__

注意这里分两种情况：

A push B

```c
1. 调用 A 的 viewWillDisappear 方法
2. 调用 B 的 viewWillAppear 方法
3. 调用 A 的 viewDidDisappear 方法
4. 调用 B 的 viewDidDAppear 方法
```

A present B

```c
1. 调用 A 的 viewWillDisappear 方法
2. 调用 C 的 viewWillAppear 方法
3. 调用 C 的 viewDidAppear 方法
4. 调用 A 的 viewDidDisappear 方法
```



# 网络



# 算法



# 待总结题目

### iOS

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
24. ARC 的本质
24. RunLoop 的基本概念，它是怎么休眠的？
24. Autoreleasepool 什么时候释放，在什么场景下使用？
24. 如何找到字符串中第一个不重复的字符
24. 哈希表如何处理冲突

### 算法

算法主要分为以下几个部分：字符串、数组与查找、链表、树以及其他基础问题。

再详细总结一下，比如递归、动态规划、BFS/DFS、双指针、二分搜索等。或者是直接考察数据结构的使用，如：哈希、栈和队列、链表等。

将阅读《剑指 Offer》列入今年的学习计划中。

列出的一些经典题目，在“【】”中标记了我对此题类型的分类，如果加星号表示此题在实际面试中出现过：

字符串

- 【3】最长回文子串（https://leetcode.com/problems/longest-palindromic-substring/）
- 【3】最长无重复子串（https://leetcode.com/problems/longest-substring-without-repeating-characters/）
- 【1*】字符串转数字（https://leetcode.com/problems/string-to-integer-atoi/）
- 【4】KMP 算法
- 【2】字符串全排列（http://blog.csdn.net/morewindows/article/details/7370155）
- 【2*】翻转字符串（https://leetcode.com/problems/reverse-words-in-a-string/）

动态规划

- 【2】背包问题（http://baike.baidu.com/link?url=rhsiQhWz0xoYrm265bZQfow1IePC8X_MVUUlw9V9Ymu9g4Cwoqu-fjZIGOJZqe_V4ONMJlXug1IN0Cj_g0O3Ya）
- 【3】连续子数组的最大和（https://leetcode.com/problems/maximum-subarray/）
- 【4】实现简单的正则表达式匹配（https://leetcode.com/problems/regular-expression-matching/）

数组

- 【3】求两个等长、有序数组的中位数（二分法）
- 【4】求两个不等长、有序数组的中位数（https://leetcode.com/problems/median-of-two-sorted-arrays/）
- 【3】旋转数组求最小值（二分法）（https://leetcode.com/problems/find-minimum-in-rotated-sorted-array/）
- 【3】旋转数组求查找某个值是否存在（二分法）（https://leetcode.com/problems/search-in-rotated-sorted-array/）
- 【4*】每行从左到右，每列从上到下递增的二维数组中，判断某个数是否存在（剑指 offer 第 3 题）
- 【3*】数组中出现次数超过一半的数字（https://bestswifter.com/arrayoccurmorethanhalf/）
- 【3*】第 k 大的数（拓展：最大的 k 个数）（https://leetcode.com/problems/kth-largest-element-in-an-array/）
- 【3*】有序数组中某个数字出现的次数（提示：利用二分搜索）

链表

- 【2】反转链表（使用递归和迭代两种解法，了解头插法）（https://leetcode.com/problems/reverse-linked-list/）
- 【3】删除链表的当前节点（https://leetcode.com/problems/delete-node-in-a-linked-list/）
- 【3】删除倒数第 k 个节点（https://leetcode.com/problems/remove-nth-node-from-end-of-list/）
- 【1】两个有序链表合并（https://leetcode.com/problems/merge-two-sorted-lists/）
- 【4】复杂链表的复制（https://leetcode.com/problems/copy-list-with-random-pointer/）
- 【2*】判断链表是否有环（https://leetcode.com/problems/linked-list-cycle-ii/）
- 【3*】两个链表的第一个公共节点（提示：考虑链表有环的情况）（http://blog.csdn.net/zzran/article/details/7984870）
- 【3】删除链表中重复节点（https://leetcode.com/problems/remove-duplicates-from-sorted-list/）

树

- 【3】根据中序和后序遍历结果重建二叉树（https://leetcode.com/problems/construct-binary-tree-from-inorder-and-postorder-traversal/）
- 【3】根据中序和前序遍历结果重建二叉树（https://leetcode.com/problems/construct-binary-tree-from-preorder-and-inorder-traversal/）
- 【2】翻转二叉树（https://leetcode.com/problems/invert-binary-tree/）
- 【2】从上往下打印二叉树 (BFS 的思想)（https://leetcode.com/problems/binary-tree-level-order-traversal-ii/）
- 【3】判断某个数组是不是二叉树的后序遍历结果 (剑指 offer 第 24 题)
- 【3】二叉树中和为某个值的路径（https://leetcode.com/problems/path-sum-ii/）
- 【3*】二叉树中某个节点的下一个节点 (强烈推荐准备一下，剑指 offer 第 58 题)

栈

- 【2】用两个栈实现队列（https://leetcode.com/problems/implement-queue-using-stacks/）
- 【2】用两个队列实现栈（https://leetcode.com/problems/implement-stack-using-queues/）
- 【2】实现一个栈，可以用常数级时间找出栈中的最小值（https://leetcode.com/problems/min-stack/）
- 【3】判断栈的压栈、弹栈序列是否合法（剑指offer 第 22 题）

排序

了解以下排序的时间、空间复杂度，是否稳定，实现原理

- 归并排序、拓展：求数组中的逆序对个数（http://baike.baidu.com/link?url=awQj_m-d5Lf_g5TglfU3uN55-0Dt2SmFv3IZx6-Ia86dOyyBoxkJhDyc2oFAzlUXFIXhs2dGehbHAHDqr1DuAK）
- 快速排序 重点：partion 函数的实现（http://baike.baidu.com/link?url=_sdhFCTSDKNM6wA5OCgqHTJc9kLWjiQBPLzO9-Cw_bYh4vZ7ofkBqySVw-MQDsJfOO9g9SJb2OCuL3yAmHIPWvv_sSPOsf-JZfKOU8QV3HrhInkkfGJ6j5Li4u3nWwiZkQlOUlGEZI-gLIyObuEM7bqwZPR8gLyVEiqutVr8pIDa6gF7cB8eixeQyur2yUt7）
- 堆排序（http://baike.baidu.com/link?url=eioUPqfLTP0mqIOIuJdH0tp9m3ihgtx2rjcFMDpKhNXcjRidBGuk-XuQtM2Zl0z6J43yKq8VTDfFRzCsN4zj3_）
- 数组元素值域已知时，考虑 基数排序 和 桶排序（http://baike.baidu.com/view/1170573.htm）（http://baike.baidu.com/view/1784217.htm）

位运算

- 【2】给一个十进制数字，求它的二进制表示中，有多少个 1 (n &= n - 1)（https://leetcode.com/problems/number-of-1-bits/）
- 【3】给一个数组，所有数字都出现了偶数次，只有一个出现了一次，找出这个数（https://leetcode.com/problems/single-number/）
- 【4】给一个数组，所有数字都出现了三次，只有一个出现了一次，找出这个数（https://leetcode.com/problems/single-number-ii/）
- 【3】给一个数组，所有数组都出现了偶数次，只有两个数字出现了一次，找出这两个数（https://leetcode.com/problems/single-number-iii/）


### 网络

根据不同的面试岗位，侧重点略有不同。对 iOS 和 Android 开发者来说，HTTP 考的略少，以 TCP 和 UDP 为主。其实 UDP 基本上只会考察和 TCP 的区别。

当然还有一些常见的基础问题，比如 Cookie 和 Session 的考察，POST 和 GET 的考察，HTTPS 的简单了解等。这些问题在我的博客中都有简单的总结。

- 【博客】我的六篇总结————不到一周（http://www.jianshu.com/notebooks/3276500/latest）
- 【书】图解 TCP/IP————半个月（http://item.jd.com/11253710.html）
- 【书】TCP/IP 详解————没读过，感觉至少需要一个月（http://item.jd.com/11182320.html）
- 【书】TCP/IP 协议簇————没读过，感觉至少需要一个月（http://item.jd.com/10485475.html）

光读书是没有用的，一问到实际问题很容易懵逼，以下是我总结的一些问题：

- 简介 TCP 和 UDP 区别，他们位于哪一层？
- 路由器和交换机的工作原理大概是什么，他们分别用到什么协议，位于哪一层？
- 描述TCP 协议三次握手，四次释放的过程。
- TCP 协议是如何进行流量控制，拥塞控制的？
- 为什么建立连接时是三次握手，两次行不行？如果第三次握手失败了怎么处理
- 关闭连接时，第四次握手失败怎么处理？
- 你怎么理解分层和协议？
- HTTP 请求中的 GET 和 POST 的区别，Session 和 Cookie 的区别。（https://bestswifter.com/bat-interview/(http://www.w3school.com.cn/tags/html_ref_httpmethods.asp)）
- 谈谈你对 HTTP 1.1，2.0 和 HTTPS 的理解。


### 操作系统与编译

我被问到的操作系统问题很少，所以仅仅总结了一些自认为比较重要的问题。关于这一部分的知识，推荐阅读《程序员的自我修养》，如果时间有限，你可以阅读我的《程序员的自我修养读书笔记》（https://bestswifter.com/cheng-xu-yuan-de-zi-wo-xiu-yang-du-shu-bi-ji/），并思考这些问题：

- 源代码是怎么变成可执行文件的，每一步的作用是什么？（预编译，词法分析，语法分析，语义分析，中间语言生成目标代码生成，汇编，链接）
- 应用层、API、运行库、系统调用、操作系统内核之间的关系是什么？
- 虚拟内存空间是什么，为什么要有虚拟内存空间。
- 静态链接和动态链接分别表示什么，大概是怎么实现的？
- 可执行文件的结构如何？（分为哪些段）
- 它是怎么装载进内存的，为什么要分段，分页，页错误是什么？
- 进程的内存格局是怎样的？（堆、栈、全局/静态区，代码区，常量区）
- 堆和栈的区别，函数调用和栈的关系
- 进程和线程的区别
- 异步和同步，串行，并发，并行的区别
- 多并发任务，仅多线程能加快速度么（不能，会变慢，有线程切换的开销）
- 多个线程之间可以共享那些数据
- 进程之间如何通信
- 介绍几种锁，他们的用途和区别

关于多线程相关的，推荐阅读这篇文章的前面一小部分——《iOS多线程编程——GCD与NSOperation总结》（https://bestswifter.com/multithreadconclusion/）

关于操作系统和编译方面的文章，除了读原书和我的读书笔记外，还可以参考这篇文章——《修改一个数字破解Mac上的应用》（https://bestswifter.com/app-crack/）

### OC

首先两本必备的神书一定是要读完的。一本是讲 OC 的《Effective Objective-C 2.0》，中文名叫：“编写高质量 iOS 与 OS X 代码的 52 个有效方法”。另一本书叫：《Objective-C 高级编程》。前者讲解 OC 中各种细节，后者主要讲了 ARC、Block 和 GCD。

光是读书，思考不够，很容易在面试时被问懵逼，所以建议一遍尝试回答面试真题，一边阅读以下总结性的文章，重要性不分先后：

- 检测内存泄露（http://wereadteam.github.io/2016/02/22/MLeaksFinder/?from=singlemessage&isappinstalled=0）
- KVO与KVC原理、KVO、Notification、Delegate优缺点、最推荐的官方文档（http://www.jianshu.com/p/f1393d10109d）（http://www.tuicool.com/articles/mUVJVb）
- GCD 与 NSOperation（https://bestswifter.com/multithreadconclusion/）
- Runtime（http://www.ianisme.com/ios/2019.html）
- block（http://blog.ibireme.com/2013/11/27/objc-block/）
- atomic 线程安全、@synchronized（http://www.cocoachina.com/bbs/read.php?tid-301674.html）（http://www.cnblogs.com/wendingding/p/3805841.html）
- 对象的深浅复制（https://bestswifter.com/differenceofcopybettenswiftandoc/）
- 招聘一个靠谱的iOS（https://github.com/ChenYilong/iOSInterviewQuestions/tree/master/01%E3%80%8A%E6%8B%9B%E8%81%98%E4%B8%80%E4%B8%AA%E9%9D%A0%E8%B0%B1%E7%9A%84iOS%E3%80%8B%E9%9D%A2%E8%AF%95%E9%A2%98%E5%8F%82%E8%80%83%E7%AD%94%E6%A1%88）
- 消息传递机制（http://objccn.io/issue-7-4/）
- 深入理解Objective-C：Category（http://tech.meituan.com/DiveIntoCategory.html）

强烈推荐第八篇文章，做完这上面的题目基本上可以应付大多数 OC 方面的问题了。

### iOS 开发

- RunLoop（http://blog.ibireme.com/2015/05/18/runloop/）
- Cell 图片异步加载优化（http://www.jianshu.com/p/02ab2b74c451）
- iOS 函数式编程的实现 && 响应式编程概念（http://www.cocoachina.com/ios/20160307/15585.html）
- 内存恶鬼drawRect（http://bihongbo.com/2016/01/03/memoryGhostdrawRect/）
- UIKit 性能调优(主要是UITableView)（https://bestswifter.com/uikitxing-neng-diao-you-shi-zhan-jiang-jie/）
- 优化UITableViewCell高度计算的那些事（http://blog.sunnyxx.com/2015/05/17/cell-height-calculation/）
- 高性能图片架构与设计（http://zhuanlan.zhihu.com/magilu/20273299）
- 轻量化视图控制器（http://objccn.io/issue-1-1/）
- UIView的生命周期（https://bestswifter.com/uiviewlifetime/）
- 高效设置圆角（https://bestswifter.com/efficient-rounded-corner/）
- 事件的传递和响应机制（http://www.jianshu.com/p/2e074db792ba）
- ReactiveCocoa 和 MVVM 入门（http://yulingtianxia.com/blog/2015/05/21/ReactiveCocoa-and-MVVM-an-Introduction/）

其中需要重点了解 runloop，它不仅仅是简单的“跑圈”的 概念，很多问题其实都与它有关，建议认真阅读 ibireme 大神的总结


### 其他面经

我是如何同时拿到阿里和腾讯offer的（https://segmentfault.com/a/1190000002627927#articleHeader5）
大三学生拿到阿里,百度实习offer面试经验分享（http://www.wtoutiao.com/p/e2fwlu.html）
2016年1月TX电面题（http://blog.csdn.net/xieyupeng520/article/details/50681147）


# 参考资料

1. [让 BAT 的 Offer 不再难拿 - bestswifter](https://bestswifter.com/bat-interview/)