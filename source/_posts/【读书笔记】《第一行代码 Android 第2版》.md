---
title: 【读书笔记】《第一行代码 Android 第2版》
date: 2018-06-23 00:05:34
categories: 读书笔记
tags:
---


> 作者：郭霖
> 
> 出版日期：2016-12-1
> 
> 豆瓣地址：https://book.douban.com/subject/26915433/
> 
> 源代码地址：https://github.com/guolindev/booksource/
> 
> Android Studio 下载地址：https://developer.android.google.cn/studio/


##  Android 简介

### 系统架构

Android 大致可以分为四层架构：Linux 内核层、系统运行库层、应用框架层、应用层。

- Linux 内核层，这一层为 Android 设备的各种硬件提供了底层的驱动，如显示驱动、音频驱动、照相机驱动、蓝牙驱动、WiFi 驱动、电源管理等。

- 系统运行库层，这一层通过一些库提供了主要的特性支持，如 SQLite 库提供了数据库、OpenGL|ES 库提供了 3D 绘图、Webkit 库提供了浏览器内核等。同样在这一层还有 Android 运行时库提供了一些核心库，允许开发者使用 Java 来编写 Android 应用、Dalvik 虚拟机针对手机内存和 CPU 性能有限等情况做了优化处理。

- 应用框架层，这一层主要提供了构建应用程序时可能用到的各种 API。

- 应用层，所有安装在手机上的应用程序都是属于这一层的，比如系统自带的联系人、短信等程序。

### 开发组件

Android 系统四大组件分别是活动（Activity）、服务（Service）、广播接收器（Broadcast Receiver）、内容提供器（Content Provider）。

- 活动，应用程序的门面，凡是在应用中看得到的东西，都是放在活动中的。

- 服务，无法看到它，但它会一直在后台默默地运行，即时用户退出了应用，服务仍然可以继续运行。

- 广播接收器，允许应用程序接收或者发送的广播消息，比如电话、短信。

- 内容提供器，提供应用程序之间数据共享，比如读取系统电话簿。


P15










