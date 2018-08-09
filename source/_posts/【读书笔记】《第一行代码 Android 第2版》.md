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

### 1. 系统架构

Android 大致可以分为四层架构：Linux 内核层、系统运行库层、应用框架层、应用层。

- Linux 内核层：这一层为 Android 设备的各种硬件提供了底层的驱动，如显示驱动、音频驱动、照相机驱动、蓝牙驱动、WiFi 驱动、电源管理等。

- 系统运行库层：这一层通过一些库提供了主要的特性支持，如 SQLite 库提供了数据库、OpenGL|ES 库提供了 3D 绘图、Webkit 库提供了浏览器内核等。同样在这一层还有 Android 运行时库提供了一些核心库，允许开发者使用 Java 来编写 Android 应用、Dalvik 虚拟机针对手机内存和 CPU 性能有限等情况做了优化处理。

- 应用框架层：这一层主要提供了构建应用程序时可能用到的各种 API。

- 应用层：所有安装在手机上的应用程序都是属于这一层的，比如系统自带的联系人、短信等程序。

### 2. 开发组件

Android 系统四大组件分别是活动（Activity）、服务（Service）、广播接收器（Broadcast Receiver）、内容提供器（Content Provider）。

- 活动：应用程序的门面，凡是在应用中看得到的东西，都是放在活动中的。

- 服务：无法看到它，但它会一直在后台默默地运行，即时用户退出了应用，服务仍然可以继续运行。

- 广播接收器：允许应用程序接收或者发送的广播消息，比如电话、短信。

- 内容提供器：提供应用程序之间数据共享，比如读取系统电话簿。


## 配置环境

### 设置夜神模拟器

1. 运行 Android Studio 和夜神模拟器
2. 打开终端，进入夜神模拟器的安装目录下，并执行连接命令

```c
$ cd /Applications/Nox\ App\ Player.app/Contents/MacOS
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

![Project 模式根目录下项目结构](http://pd5z3o7sm.bkt.clouddn.com/image/2018-06-23/001.png)

#### 1. .gradle 和 .idea

这两个目录下放置的都是 Android Studio 自动生成的一些文件，无需关心，也不要手动编辑。

#### 2. app

项目中的代码、资源等内容几乎都是放置在这个目录下的，我们后面的开发工作也基本都是在这个目录下进行的，下面会详细展开讲解。

#### 3. gradle

这个目录下包含了 gradle wrapper 的配置文件，使用 gradle wrapper 的方式不需要提前将 gradle 下载好，而是会自动根据本地的缓存情况决定是否需要联网下载 gradle。Android Studio 默认没有启用 gradle wrapper 的方式，如果需要打开，可以点击 Android Studio 导航栏 -> File -> Settings -> Build,Execution,Deployment -> Gradle 进行配置更改。

#### 4. build.gradle

这是项目全局的 gradle 构建脚本，通常这个文件中的内容是不需要修改的，除非你想添加一些全局的项目构建配置。不同于 Eclipse，Android Studio 是采用 Gradle 来构建项目的。 代码如下所示：

```java
buildscript {
    
    repositories {
        google()
        jcenter()
    }
    dependencies {
        classpath 'com.android.tools.build:gradle:3.1.3'
    }
}

allprojects {
    repositories {
        google()
        jcenter()
    }
}

task clean(type: Delete) {
    delete rootProject.buildDir
}
```

首先，两处 `repositories` 闭包中都声明了 `jcenter()` 这行配置，这是一个代码托管仓库，配置之后可以在项目中轻松引用任何 jcenter 上的开源项目了。

接下来 `dependencies` 闭包中使用 `classpath` 声明了一个 Gradle 插件，因为 Gradle 并不是专门为构建 Android 项目而开发的，Java、C++ 等很多项目都可以使用 Gradle 来构建。因此如果我们想使用它来构建 Android 项目，则需要声明 `'com.android.tools.build:gradle:3.1.3'` 这个插件。

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

![Project 模式 app 目录下项目结构](http://pd5z3o7sm.bkt.clouddn.com/image/2018-06-23/002.png)

#### 1. build

这个目录主要包含了一些在编译时自动生成的文件，我们不需要过多关心。

#### 2. libs

如果你的项目中使用了第三方 jar 包，就需要把这些 jar 包都放在 libs 目录下，并且会被自动添加到构建路径里去。

#### 3. androidTest

此处是用来编写 Android Test 测试用例的，可以对项目进行一些自动化测试。

#### 4. java

毫无疑问，java 目录是放置我们所有 Java 代码的地方。

#### 5. res

项目中使用到的所有图片、布局、字符串等资源都要存放在这个目录下。图片放在 drawable 目录下、图标放在 mipmap 目录下、布局放在 layout 目录下、字符串放在 values 目录下。

#### 6. AndroidManifest.xml

这是整个 Android 项目的配置文件，在程序中定义的所有四大组件都需要在这个文件里注册，另外还可以在这个文件中给应用程序添加权限声明。

#### 7. test

此处是用来编写 Unit Test 测试用例的，是对项目进行自动化测试的另一种方式。

#### 8. app.iml

IntelliJ IDEA 项目自动生成的文件，我们不需要关系或修改这个文件中的内容。

#### 9. build.gradle

这是 app 模块的 gradle 构建脚本，这个文件中会指定很多项目构建相关的配置。代码如下所示：

```java
apply plugin: 'com.android.application'

android {
    compileSdkVersion 28
    defaultConfig {
        applicationId "com.example.mayan.helloworld"
        minSdkVersion 15
        targetSdkVersion 28
        versionCode 1
        versionName "1.0"
        testInstrumentationRunner "android.support.test.runner.AndroidJUnitRunner"
    }
    buildTypes {
        release {
            minifyEnabled false
            proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'
        }
    }
}

dependencies {
    implementation fileTree(dir: 'libs', include: ['*.jar'])
    implementation 'com.android.support:appcompat-v7:28.0.0-alpha3'
    implementation 'com.android.support.constraint:constraint-layout:1.1.2'
    testImplementation 'junit:junit:4.12'
    androidTestImplementation 'com.android.support.test:runner:1.0.2'
    androidTestImplementation 'com.android.support.test.espresso:espresso-core:3.0.2'
}
```

首先第一行应用了一个插件，一般有两种值可选：`'com.android.application'` 表示这是一个应用程序模块，`'com.android.library'` 表示这是一个库模块。应用程序模块可以直接运行，而库模块只能作为代码库依附于别的应用程序模块来运行。

接下来是一个大的 `android` 闭包，在这个闭包中我们可以配置项目构建的各种属性。其中 `compileSdkVersion` 用于指定项目的编译版本。

在 `android` 闭包中又嵌套了一个 `defaultConfig` 闭包，其中可以对项目的更多细节进行配置。其中 `applicationId` 用于指定项目的包名，如果后续想对其修改，那么就是在这里修改。`minSdkVersion` 用于指定项目最低兼容的 Android 系统版本，这里 15 表示最低兼容到 Android 4.0 系统。`targetSdkVersion` 表示在目标版本上已经做过了充分的测试，系统将会为应用启用一些最新的功能和特性。比如说 Android 6.0 系统中引入了运行时权限这个功能，如果将 `targetSdkVersion` 指定成 23 或者更高，系统就会为你的程序启用运行时权限功能，而如果你将 `targetSdkVersion` 指定成 22，那么就说明程序最高只在 Android 5.1 系统上做过充分的测试，Android 6.0 系统中引入的新功能自然就不会启用了。最后 `versionCode` 用于指定项目的版本号，`versionName` 用于指定项目的版本名。

接下来我们看一下 `buildTypes` 闭包，其中用于指定生成安装文件的相关配置，通常只会有两个子闭包，一个是 `debug`，一个是 `release`，其中 `debug` 闭包是可以忽略不写的。`release` 闭包中，`minifyEnabled` 用于指定是否对项目的代码进行混淆，`proguardFiles` 用于指定混淆时使用的规则文件，第一个文件 `'proguard-android.txt'` 是在 Android SDK 目录下的，里面是所有项目通用的混淆规则，第二个文件 `'proguard-rules.pro'` 是在当前项目的根目录下的，里面可以编写当前项目特有的混淆规则。

最后 `dependencies` 闭包可以指定当前项目所有的依赖关系，通常有三种依赖方式：本地依赖、库依赖、远程依赖。本地依赖可以对本地的 Jar 包或者目录添加依赖关系，库依赖可以对项目中的库模块添加依赖关系，远程依赖可以对 jcenter 库上的开源项目添加依赖关系。 

#### 10. proguard-rules.pro

这个文件用于指定项目代码的混淆规则，当代码开发完成后打成安装包文件，如果不希望代码被别人破解，通常会将代码进行混淆，从而让破解者难以阅读。

### 日志工具 Log

Android 中日志工具类 Log 提供了五种方法来打印日志：

- `Log.v()` 用于打印那些最为琐碎的、意义最小的日志信息。对应级别 verbose，是 Android 日志里面级别最低的一种。
- `Log.d()` 用于打印一些调试信息。对应级别 debug，比 verbose 高一级。
- `Log.i()` 用于打印一些比较重要的数据，可以帮助分析用户行为数据。对应级别 info，比 debug 高一级。
- `Log.w()` 用于打印一些警告信息，提示程序在这个地方可能会有潜在的风险，最好去修复一下这些出现警告的地方，对应级别 warn，比 info 高一级。
- `Log.e()` 用于打印程序中的错误信息，比如程序进入到了 catch 语句当中，当有错误信息打印出来的时候，一般都代表程序出现严重问题了，必须尽快修复。对应级别 error，比 warn 高一级。


## 活动

### 手动创建活动

#### 创建活动

新创建一个 Android 项目，项目名称可以叫做 ActivityTest，在添加活动页面选择 Add No Activity，默认创建一个没有 Activity 的项目。

![Add No Activity](http://pd5z3o7sm.bkt.clouddn.com/image/2018-06-23/003.png)

目前项目中虽然还是会自动生成很多文件，但是 app/src/main/java/com.example.mayan.activitytest 目录应该是空的。右击 com.example.mayan.activitytest 包 -> New -> Activity -> Empty Activity，将活动命名为 FirstActivity。

![Empty Activity](http://pd5z3o7sm.bkt.clouddn.com/image/2018-06-23/004.png)

其中勾选 `Generate Layout File` 表示会自动为 FirstActivity 创建一个对应的布局文件，勾选 `Launcher Activity` 表示会自动将 FirstActivity 设置为当前项目的主活动，勾选 `Backwards Compatibility` 表示会为项目启用向下兼容的模式。不要勾选 `Generate Layout File` 和 `Launcher Activity` 这两个选项，下面我们将会一个个手动来完成。

#### 创建和加载布局

Android 程序的设计讲究逻辑和视图分离，最好每一个活动都能对应一个布局，布局就是用来显示界面内容的，因此我们现在就来手动创建一个布局文件。

右击 app/src/main/res 目录 -> New -> Directory，创建一个名为 layout 的目录。然后右击 layout目录 -> Layout resource file，创建一个名为 first_layout 的布局文件，根元素默认选择为 LinearLayout。我们现在对这个布局稍作编辑，添加一个按钮：

```java
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:orientation="vertical" android:layout_width="match_parent"
    android:layout_height="match_parent">

    <!--match_parent 表示当前元素和父元素一样宽-->
    <!--wrap_content 表示当前高度只要能刚好包含里面的内容就好-->
    <Button
        android:id="@+id/button_1"
        android:layout_width="match_parent"   
        android:layout_height="wrap_content"  
        android:text="Button 1"
        />

</LinearLayout>
```

然后重回 FirstActivity 活动中加载这个布局：

```java
public class FirstActivity extends AppCompatActivity {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.first_layout);
    }
}
```

#### 在 AndroidManifest 文件中注册

虽然 Android Studio 已经在 AndroidManifest.xml 中自动注册了活动，但是还没有为程序配置主活动。







