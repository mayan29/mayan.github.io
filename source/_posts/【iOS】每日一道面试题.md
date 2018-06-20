---
title: 【iOS】每日一道面试题
date: 2018-06-19 10:21:55
categories: iOS
tags:
---

坚持每日记录一道面试题，持续更新。★ 为超级简单题目，★★ 为简单题目，以此类推，满级 ★★★★★ 五星。

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

> 用加法或者异或都可以，我比较偏好于异或，使用加法，如果两个数值太大，容易超出 int 范围。


## 3

__题目：OC 中 load 和 initialize 方法有何异同？__

__星级：★__ 


## 参考资料

1. [让 BAT 的 Offer 不再难拿 - bestswifter](https://bestswifter.com/bat-interview/)