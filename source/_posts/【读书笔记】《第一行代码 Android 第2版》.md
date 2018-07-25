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


## Android 简介

### 系统架构

Android 大致可以分为四层架构：Linux 内核层、系统运行库层、应用框架层、应用层。

- Linux 内核层：这一层为 Android 设备的各种硬件提供了底层的驱动，如显示驱动、音频驱动、照相机驱动、蓝牙驱动、WiFi 驱动、电源管理等。

- 系统运行库层：这一层通过一些库提供了主要的特性支持，如 SQLite 库提供了数据库、OpenGL|ES 库提供了 3D 绘图、Webkit 库提供了浏览器内核等。同样在这一层还有 Android 运行时库提供了一些核心库，允许开发者使用 Java 来编写 Android 应用、Dalvik 虚拟机针对手机内存和 CPU 性能有限等情况做了优化处理。

- 应用框架层：这一层主要提供了构建应用程序时可能用到的各种 API。

- 应用层：所有安装在手机上的应用程序都是属于这一层的，比如系统自带的联系人、短信等程序。

### 开发组件

Android 系统四大组件分别是活动（Activity）、服务（Service）、广播接收器（Broadcast Receiver）、内容提供器（Content Provider）。

- 活动：应用程序的门面，凡是在应用中看得到的东西，都是放在活动中的。

- 服务：无法看到它，但它会一直在后台默默地运行，即时用户退出了应用，服务仍然可以继续运行。

- 广播接收器：允许应用程序接收或者发送的广播消息，比如电话、短信。

- 内容提供器：提供应用程序之间数据共享，比如读取系统电话簿。


## 运行 Hello World

### 设置夜神模拟器

1. 运行 Android Studio 和夜神模拟器
2. 打开终端，进入夜神模拟器的安装目录下，并执行连接命令

```c
$ cd /Applications/Nox\ App\ Player.app/Contents/MacOS`
$ adb connect 127.0.0.1:62001
```

### 设置模拟器可能出现的问题

#### 1. `adb: command not found` 问题

- 打开终端，输入 `cd ~/` 进入当前用户的 home 目录；
- 输入 `touch .bash_profile` 如果没有 .bash_profile 文件，则创建一个该文件；
- 输入 `open .bash_profile` 打开该文件，如果是第一次配置环境，那么文本编辑框为空白；
- 在打开的文本编辑器中写入如下代码，其中 ANDROID_HOME 的 sdk 路径可以打开 preference 中搜索 sdk 来查看；

```c
export ANDROID_HOME=/Users/mayan/Library/Android/sdk
export PATH=${PATH}:${ANDROID_HOME}/tools
export PATH=${PATH}:${ANDROID_HOME}/platform-tools
```

- 在终端中输入 `source .bash_profile` 使改动生效；
- 输入 `adb` 验证是否完成配置，如果不显示 `adb: command not found` 则说明配置完成。

#### 2. 点击运行按钮后找不到夜神模拟器

重启 Android Studio 和夜神模拟器。

### 分析项目结构

点击项目目录结构上面的 Android 区域切换成 Project 模式，这就是项目真实的目录结构了。

#### 1. .gradle 和 .idea

这两个目录下放置的都是 Android Studio 自动生成的一些文件，无需关心，也不要手动编辑。

#### 2. app

项目中的代码、资源等内容几乎都是放置在这个目录下的，我们后面的开发工作也基本都是在这个目录下进行的，下面会详细展开讲解。

#### 3. gradle

这个目录下包含了 gradle wrapper 的配置文件，使用 gradle wrapper 的方式不需要提前将 gradle 下载好，而是会自动根据本地的缓存情况决定是否需要联网下载 gradle。Android Studio 默认没有启用 gradle wrapper 的方式，如果需要打开，可以点击 Android Studio 导航栏 -> File -> Settings -> Build,Execution,Deployment -> Gradle 进行配置更改。

#### 4. build.gradle

这是项目全局的 gradle 构建脚本，通常这个文件中的内容是不需要修改的。

#### 5. gradle.properties

这个文件是全局的 gradle 配置文件，在这里配置的属性将会影响到项目中所有的 gradle 编译脚本。

#### 6. gradlew 和 gradlew.bat

这两个文件是用来在命令行界面中执行 gradle 命令的，其中 gradlew 是在 Linux 或 Mac 系统中使用的，gradlew.bat 是在 Windows 系统中使用的。

#### 7. HelloWorld.iml

iml 文件是所有 IntelliJ IDEA 项目都会自动生成的一个文件（Android Studio 是基于 IntelliJ IDEA 开发的），用于标识这是一个 IntelliJ IDEA 项目，我们不需要修改这个文件中任何内容。

#### 8. local.properties

这个文件用于指定本机中的 Android SDK 路径，通常内容都是自动生成的，我们并不需要修改，除非本机中的 Android SDK 位置发生了变化。

#### 9. settings.gradle

这个文件用于指定项目中所有引入的模块，由于 HelloWorld 项目中就只有一个 app 模块，因此该文件中也就只引入了 app 这一个模块。通常模块的引入都是自动完成的，需要我们手动去修改这个文件的场景可能比较少。

### 分析 app 目录下项目结构

#### 1. build

这个目录和外层的 build 目录类似，主要也是包含了一些在编译时自动生成的文件，不过它里面的内容会更多更杂，我们不需要过多关心。

#### 2. libs

如果你的项目中使用了第三方 jar 包，就需要把这些 jar 包都放在 libs 目录下，并且会被自动添加到构建路径里去。

P19

