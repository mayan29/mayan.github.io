---
title: 【iOS】每日一道面试题
date: 2018-06-19 10:21:55
categories: iOS
tags:
---

坚持每日记录一道面试题，持续更新。★ 为超级简单题目，★★ 为简单题目，以此类推，满级 ★★★★★ 五星。


# 1

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


# 2

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


# 3

__题目：OC 中 load 和 initialize 方法有何异同？__
__星级：★__

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


# 参考资料

1. [让 BAT 的 Offer 不再难拿 - bestswifter](https://bestswifter.com/bat-interview/)