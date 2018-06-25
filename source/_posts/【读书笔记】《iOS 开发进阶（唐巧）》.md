---
title: 【读书笔记】《iOS 开发进阶（唐巧）》
date: 2017-04-27 16:16:55
categories: 读书笔记
tags:
---


> 作者：唐巧
> 
> 出版日期：2015-1-1
> 
> 豆瓣地址：https://book.douban.com/subject/26287173/


## 一、CocoaPods 的安装和使用

### 1. 简介

每种语言发展到一定阶段，都会出现相应的依赖管理工具，例如 Java 的 Maven、Node.js 的 npm 等。CocoaPods 是开发 iOS 应用程序的一个第三方库的依赖管理工具，起始于 2011 年 8 月，用 Ruby 写的。在没有使用 CocoaPods 以前，我们需要:

1. 把这些第三方开源库的源代码文件复制到项目中，或者设置成 git 的 `submodule`。
2. 对于某些开源库，我还需要设置 -licucore 或者 -fno-objc-arc 等编译参数
3. 管理这些依赖包的更新。

这些体力活虽然简单，但毫无技术含量并且浪费时间。在使用 CocoaPods 之后，就会自动将这些第三方开源库的源码下载下来，并且为我的工程设置好相应的系统依赖和编译参数。

### 2. 原理

CocoaPods 的原理是将所有的依赖库都放到另一个名为 Pods 的项目中，然后让主项目依赖 Pods 项目。下面是一些技术细节：

1. Pods 项目最终会编译成一个名为 libPods.a 的文件，主项目只需要依赖这个 .a 文件即可
2. 对于资源文件，CocoaPods 提供了一个名为 Pods-resources.sh 的 bash 脚本，该脚本每次项目编译时都会执行，将第三方库的各种资源文件复制到目标目录中
3. CocoaPods 通过一个名为 Pods.xcconfig 的文件在编译时设置所有的依赖和参数 

### 3. 安装

Mac 自带 ruby，如果 ruby 的开发工具包 gem 太老，首先升级 gem

```
$ sudo gem update --system
```

ruby 的软件源 rubygems.org 因为使用亚马逊的云服务，所以被墙了，需要更新一下 ruby 的源，下面代码将官方的 ruby 源替换成国内淘宝的源：

```
$ gem sources --remove https://rubygems.org/ 
$ gem sources -a https://ruby.taobao.org/
```

验证 ruby 的源是否为淘宝镜像

```
$ gem sources -l
```

使用 ruby 的 gem 命令即可下载安装：

```
$ sudo gem install cocoapods
$ pod setup
```

执行 pod setup 时，会输出 Setting up CocoaPods master repo 并等待好久，此时是 CocoaPods 将它的信息下载到 ~/.cocoapods 目录下。可以 cd 到那个目录，用 du -sh 来查看下载进度。

如果没有进度，那还需更换 ruby 镜像

```
$ gem sources --remove https://ruby.taobao.org/ 
$ gem sources -a https://gems.ruby-china.org/
```

更换成功后再次下载安装

```
$ sudo gem install cocoapods
$ pod setup
```

### 4. 使用

在项目文件夹中创建名为 Podfile 的文件

```
$ touch Podfile
```

在文件中添加依赖库名称

```
platform :ios, ‘8.0’

target ‘jiaheyingyuan’ do
pod ‘AFNetworking’
pod ‘SDWebImage’
end
```

如果不确定三方库版本，查找第三方库

```
$ pod search AFNetworking
```

然后在项目文件夹中执行

```
$ pod install
```

注意事项

- 每次更改了 Podfile 文件，都需要重新执行一次 pod update 命令
- 执行 pod install 之后，会生成一个名为 Podfile.lock 的文件，不能把这个文件加入到 gitignore 中，因为 Podfile.lock 会锁定当前各依赖库的版本，之后执行 pod install 也不会更改版本，只有执行 pod update 才会改变 Podfile.lock。这样可以防止第三方库升级时造成大家各自的第三方库版本不一致。
 

### 5. 其他

#### 为自己的项目创建 podspec 文件

参考以下两篇文章：

- [《如何编写一个 CocoaPods 的 spec 文件》](http://ishalou.com/blog/2012/10/16/how-to-create-a-cocoapods-spec-file/)
- [《Cocoapods 入门》](http://studentdeng.github.io/blog/2013/09/13/cocoapods-tutorial/)

#### 使用私有的 pods

我们可以直接指定某一个依赖的 podspec，这样就可以使用企业内部的私有库。该方案有利于使企业内部的公共项目支持 CocoaPods，例：

```
$ pod 'MyCommon', :podspec => 'https://yuantiku.com/common/myCommon.podspec'
```

#### 不更新 podspec

在执行 pod install 和 pod update 时，默认先更新一次 podspec 索引，如下代码可以禁止其做索引更新操作

```
$ pod install --no-repo-update
$ pod update  --no-repo-update
```

#### 生成第三方库的帮助文档

如果你想让 CocoaPods 帮你生产第三方库的帮助文档，并集成到 Xcode 中，那么用 brew 安装 appledoc 即可：

```
$ brew install appledoc
```


## 二、网络封包分析工具 Charles

![image016](/img/img016.png)

### 1. 功能简介

- 支持 SSL 代理
- 支持流量控制，可以模拟慢速网络、等待时间较长的请求
- 支持 AJAX 调试，自动将 JSON 或 XML 数据格式化，方便查看
- 支持 AMF 调试，将 Flash Remoting 或 Flex Remoting 信息格式化，方便查看
- 支持重发网络请求，方便后端调试
- 支持修改网络请求参数
- 支持网络请求的截获和动态修改
- 检查 HTML、CSS、RSS 内容是否符合 W3C 标准（http://validator.w3.org）

### 2. 安装和使用

#### 安装 Charles

在 Charles 官方网站（https://www.charlesproxy.com）下载安装。

#### 安装 SSL 证书

如果需要截取分析 SSL 协议相关内容，那么需要安装 Charles 的 CA 证书（http://www.charlesproxy.com/ssl.zip）。解压后双击 .crt 文件，在钥匙串 -> 系统 -> 证书中可查看

#### 截取 Mac 上的网络封包

Charles 是通过将自己设置成代理服务器来完成封包截取的，将 Charles 设置成系统代理，菜单中 Proxy -> Mac OS X Proxy 将 Charles 设置成系统代理。之后浏览网页就可以看到网络请求出现在 Charles 界面中。

#### 过滤网络请求

- 在主界面的中部的 Filter 栏中填入需要过滤出来的关键字，比如 baidu。这种方法是临时性的封包过滤。

- Proxy -> Recording Settings -> Include，选择添加一个项目，然后填入需要监控的协议、主机地址、端口号。这种方法是经常性的封包过滤。

#### 截取 iPhone 上的网络封包

Proxy -> Proxy Settings 填入代理端口 8888，并勾选 Enable transparent HTTP proxying 然后在手机端设置代理即可

#### 模拟慢速网络

Proxy -> Throttle Setting 勾选 Enable Throttling 并且可以设置 Throttle Preset 的类型。如果只想模拟指定网站的慢速网络，可以勾选 Only for selected hosts 然后在对话框的下半部分设置中增加指定的 Hosts 项即可。

#### 截取 SSL 信息

在该请求上单机右键，选择 SSL Proxying 然后对于该 Host 的所有 SSL 请求都可以被截取到了

#### 修改网络请求内容

调试接口时，我们需要反复尝试不同参数的网络请求，在网络请求上点击右键，选择 Edit 即可创建一个可编辑的网络请求，可修改 URL 地址、端口、参数等，修改完成后单击 Execute 即可发送

#### 修改服务器返回内容

Charles 提供了 Map、Rewrite、Breakpoints 功能，都可以达到修改服务器返回内容的目的。

- Map 适合长期将某一些请求重定向到另一个网络地址或本地文件
- Rewrite 适合对网络请求进行一些正则替换
- Breakpoints 适合做一些临时性的修改

#### Map 功能

Map 功能分为两种，进入方式 Tools -> Map Remote 或 Map Local 

- Map Remote：将指定的网络请求重定向到另一个网址
- Map Local：将指定的网络请求重定向到本地文件

对于 Map Remote 功能，我们需要分别填写网络重定向的源地址和目的地址，对于不需要限制的条件，可以留空。举例：Map From 测试服务器的请求重定向到 Map To 线上服务器。

对于 Map Local 功能，我们需要填写重定向的源地址和本地的目标文件。对于一些复杂的网络请求结果，我们可以先使用右键单击 Save Response 功能，将请求结果保存到本地，然后稍加修改，使其成为我们的目标映射文件。

#### Rewrite 功能

Rewrite 功能适合对某一类网络请求进行一些替换，以达到修改结果的目的。例如，一个 API 请求时获得用户昵称，将 Rewrite Rule 界面的 Match 的 value 设置之前的昵称，Replace 的 value 设置将要修改的昵称

#### Breakpoints 功能

Rewrite 功能适合做批量和长期的替换，临时性的修改最好使用 Breakpoints 功能。Breakpoints 功能类似 Xcode 中设置的断点，当网络请求发生时，Charles 会截取该请求，这个时候，我们可以在 Charles 中临时修改网络请求的返回内容。修改完成后单击 Execute 即可让网络请求继续进行。需要注意的是，使用 Breakpoints 功能将网络请求截取并修改的过程中，整个网络请求的计时并不会暂停，所以长时间的暂停可能导致客户端的请求超时。


## 三、其他一些实用的工具


### 1. 界面调试工具 Reveal

[Reveal](http://revealapp.com) 可以在 iOS 开发时动态的查看和修改应用程序的界面。iOS 逆向工程中使用强大，可以分析他人的 APP，缺点是真特么贵。

![image017](/img/img017.png)

### 2. 移动统计工具 Flurry

[Flurry](http://www.flurry.com) 是一家专门为移动应用提供数据统计和分析的公司，使用 Flurry 服务的公司包括 eBay、Yahoo、Skype 等超过 11 万家公司。Flurry 优点：保持独立和专注，数据安全性更高。友盟已经被阿里收购，当用户的应用涉及的业务和阿里有类似或重合的时候，该统计数据有潜在的安全性问题

### 3. 崩溃日志记录工具 Crashlytics

[Crashlytics](http://try.crashlytics.com) 是专门为移动应用开发者提供的保存和分析应用崩溃信息的专业工具。

### 4. App Store 统计工具 App Annie

[App Annie](http://www.appannie.com) 是一个 App Store 数据的统计分析工具。该工具可以统计 App 在 App Store 的下载量、排名变化、销售收入情况、用户评价等信息。

苹果官方的 iTunes Connect 提供的销售数据统计功能比较差，例如只能保存最近30天的详细销售数据、界面丑陋、无法查看应用的排名历史变化情况等

### 5. Xcode 插件

- Alcatraz：管理 Xcode 插件、模板和颜色配置
- KSImageNamed：自动弹出图片预览
- BBUDebuggerTuckAway：智能弹出隐藏调试窗口
- SCXcodeSwitchExpander：迅速地在 switch 语句中填充枚举类型的每种可能的取值
- VVDocumenter：自动生成代码注释
- ClangFormat：自动调整代码风格
- ColorSense：编写 UIColor 时，实时预览相应的颜色

### 6. Dash

Dash 是一款 API 文档查询及代码片段管理工具，超级好用

### 7. Appledoc

Appledoc 是一个从源代码中抽取文档的工具


## 四、理解内存管理

### 1. 什么是引用计数？

引用计数可以有效地管理对象的生命周期。当我们创建一个新对象的时候，它的引用计数为 1，当有一个新的指针指向这个对象时，我们将其引用计数加 1，当某个指针不再指向这个对象时，我们将其引用计数减 1，当对象的引用计数变为 0 时，说明这个对象不再被任何指针指向了，这个时候我们就可以将对象销毁，回收内存。

### 2. 我们为什么需要引用计数

假如对象 A 将其中的对象 M 作为参数传递给对象 B，没有引用计数的情况下，内存管理原则是“谁申请谁释放”。那么要在 B 不再需要 M 的时候，A 将 M 销毁。但 B 可能只是临时用一下 M，也可能觉得 M 很重要，将它设置成自己的一个成员变量。这种情况下，什么时候销毁 M 就成了一个难题。

有一个暴力的做法，就是 A 调用完 B 之后，马上就销毁参数 M，然后 B 将参数另外复制一份 M2，自己管理 M2 的生命期。这种做法有一个很大的问题，就是它带来了更多的内存申请、复制、释放的工作，实在太影响性能。

还有另外一种做法，A 在构造完 M 之后，始终不销毁 M，由 B 来完成 M 的销毁工作。如果 B 需要长时间使用 M，就不销毁它，如果只是临时用一下，则可以用完马上销毁。这样好像很好的解决了对象复制的问题，但是它强烈依赖于 A、B 两个对象的配合。而且 M 申请在 A 中，释放在 B 中，使得它的内存管理代码分散在不同对象中，管理起来非常费劲。再复杂点，B 需要再向 C 传递 M，那么 M 在 C 中又不能让 C 管理，所以这种方式带来的复杂性更大。

所以引用计数很好的解决了这个问题，哪些对象需要长时间使用，就把它的引用计数加1，使用完了再把引用计数减1，对象的生命期管理可以完全交给引用计数了。

### 3. 不要向已经释放的对象发送消息

```objc
NSObject *obj = [[NSObject alloc] init];
NSLog(@"Reference Count = %u", [obj retainCount]);
[obj release];
NSLog(@"Reference Count = %u", [obj retainCount]);
```

输出结果可能是这样的

```objc
Reference Count = 1
Reference Count = 1
```

最后一次输出，引用计数为什么没有变成 0 呢？因为该对象的内存已经被回收，而我们向一个已经被回收的对象发了一个 retainCount 消息，所以它的输出结果应该是不确定的，如果该对象所占的内存被复用了，那么就有可能造成程序异常崩溃。

那为什么在这个对象被回收之后，这个不确定的值是 1 而不是 0 呢？因为当最后一次执行 release 时，系统知道马上就要回收内存了，就没有必要将 retainCount 减 1 了，因为不管减不减 1，该对象都肯定被回收，而对象被回收后，它所有的内存区域，包括 retainCount 值也变得没有意义。这样减少一次内存的操作，加速对象的回收。

### 4. 弱引用

弱引用持有对象，但是不增加引用计数，这样就避免了循环引用的产生

### 5. Core Foundation 对象的内存管理

```objc
CFStringRef str = CFStringCreateWithCString(kCFAllocatorDefault, "hello world", kCFStringEncodingUTF8);
    
CFRetain(str);  // 引用计数 +1
CFRelease(str);  // 引用计数 -1
```

CFRetain 和 CFRelease 方法与 Objective-C 对象的 retain 和 release 方法等价。

将 Core Foundation 对象转换成一个 Objective-C 对象，引入了 bridge 相关关键字：

- `__bridge`：只做类型转换，不修改相关对象的引用计数，原来的 Core Foundation 对象在不用时，需要调用 CFRelease 方法
- `__bridge_retained`：类型转换后，将相关对象的引用计数加 1，原来的 Core Foundation 对象在不用时，需要调用 CFRelease 方法
- `__bridge_transfer`：类型转换后，将该对象的引用计数交给 ARC 管理，Core Foundation 对象在不用时，不再需要调用 CFRelease 方法


## 五、后台运行

正常情况下，当应用按 Home 键退出后，应用仅有最多 5 秒时间做一些保存后请理资源的工作。但是应用可以调用后台方法，让应用最多有 10 分钟的时间在后台长久运行。

```objc
@property (nonatomic, assign) UIBackgroundTaskIdentifier backgroundUpdateTask;

- (void)applicationDidEnterBackground:(UIApplication *)application {
    
    [self beingBackgroundUpdateTask];
    // 在这里加上你需要长久运行的代码
    [self endBackgroundUpdateTask];
}

- (void)beingBackgroundUpdateTask {
 
    self.backgroundUpdateTask = [[UIApplication sharedApplication] beginBackgroundTaskWithExpirationHandler:^{
        [self endBackgroundUpdateTask];
    }];
}

- (void)endBackgroundUpdateTask {
 
    [[UIApplication sharedApplication] endBackgroundTask:self.backgroundUpdateTask];
    self.backgroundUpdateTask = UIBackgroundTaskInvalid;
}
```


## 六、UIWindow 应用场景

### 1. WindowLevel

UIWindow 有一个类型为 `UIWindowLevel` 的属性，该属性定义了 UIWindow 的层级，系统定义的一共有 3 种取值：

```objc
UIWindowLevelNormal    = 0 
UIWindowLevelStatusBar = 1000
UIWindowLevelAlert 	   = 2000
```

默认程序的 UIWindow 的层级就是 UIWindowLevelNormal，当系统需要在其上面覆盖 UIAlertView 时，就会创建一个层级为 UIWindowLevelAlert 的 UIWindow。在实际应用中，WindowLevel 的取值并不限于这 3 个值。

### 2. 使用场景

适合用 UIWindow 来实现的功能包括：手势解锁、启动页、通知提醒显示、弹框广告等。可以封装成一个单例，方便调用：

```objc
#import <UIKit/UIKit.h>

@interface MYAlertWindow : UIWindow

+ (instancetype)sharedInstance;

- (void)show;

@end
```

```objc
#import "MYAlertWindow.h"

@implementation MYAlertWindow

+ (instancetype)sharedInstance {
    
    static id sharedInstance = nil;
    static dispatch_once_t onceToken;
    dispatch_once(&onceToken, ^{
        sharedInstance = [[self alloc] initWithFrame:[UIScreen mainScreen].bounds];
    });
    return sharedInstance;
}

- (instancetype)initWithFrame:(CGRect)frame
{
    self = [super initWithFrame:frame];
    if (self) {
        
        self.backgroundColor = [UIColor grayColor];
        
        UILabel *label = [[UILabel alloc] initWithFrame:CGRectMake(0, 0, 200, 100)];
        label.center = self.center;
        label.backgroundColor = [UIColor whiteColor];
        label.text = @"警告";
        label.textAlignment = NSTextAlignmentCenter;
        [self addSubview:label];
        
        UITapGestureRecognizer *tap = [[UITapGestureRecognizer alloc] initWithTarget:self action:@selector(tapClick)];
        [self addGestureRecognizer:tap];
    }
    return self;
}

- (void)tapClick {
    [self resignKeyWindow];
    self.hidden = YES;
}

- (void)show {
    [self makeKeyWindow];
    self.hidden = NO;
}

@end
```

调用下列方法即可使用：

```objc
[[MYAlertWindow sharedInstance] show];
```


## 七、动态下载系统提供的多种中文字体

字体文件通常比较大，10 ~ 20 MB 是常见的字体库的大小，如果只是很少量的字体需要设置，导入整个字体库是不划算的，并且中文字体通常都是有版权的，所以使用特殊中文字体库的 iOS 应用较少，通常只有阅读类的应用才会使用特殊中文字体库。从 iOS 6 开始，苹果支持动态下载中文字体到系统中，使用系统提供的中文字体，既可以避免版权问题，又可以减少应用体积。

首先需要使用 Mac 内自带的应用“字体册”（Font Book）来获得相应字体的 PostScript 名称。

假如我们现在要下载“娃娃体”，它的 PostScript 名称为“DFWaWaSC-W5”，首先判断该字体是否已经被下载下来

```objc
- (BOOL)isfontDownloaded:(NSString *)fontName
{
    UIFont *aFont = [UIFont fontWithName:fontName size:12.0];
    if (aFont && ([aFont.fontName compare:fontName] == NSOrderedSame || [aFont.familyName compare:fontName] == NSOrderedSame)) {
        return YES;
    } else {
        return NO;
    }
}
```

如果该字体没有下载过，我们需要准备下载字体 API 需要的一些参数

```objc
NSMutableDictionary *attrs = [NSMutableDictionary dictionaryWithObjectsAndKeys:fontName, kCTFontNameAttribute, nil];
    
// 创建一个字体描述对象
CTFontDescriptorRef desc = CTFontDescriptorCreateWithAttributes((__bridge CFDictionaryRef)attrs);
    
NSMutableArray *descs = [NSMutableArray arrayWithObject:(__bridge id _Nonnull)(desc)];
CFRelease(desc);
```

准备好上面的 descs 变量后，就可以进行字体的下载了

```objc
__block BOOL errorDuringDownload = NO;
CTFontDescriptorMatchFontDescriptorsWithProgressHandler((__bridge CFArrayRef)descs, NULL, ^bool(CTFontDescriptorMatchingState state, CFDictionaryRef  _Nonnull progressParameter) {
        
    double progressValue = [[(__bridge NSDictionary *)progressParameter objectForKey:(id)kCTFontDescriptorMatchingPercentage] doubleValue];
        
    if (state == kCTFontDescriptorMatchingDidBegin) {
        NSLog(@"字体已经匹配");
    } else if (state == kCTFontDescriptorMatchingDidFinish) {
        if (!errorDuringDownload) {
            NSLog(@"字体 %@ 下载完成", fontName);
        }
    } else if (state == kCTFontDescriptorMatchingWillBeginDownloading) {
       NSLog(@"字体开始下载");
    } else if (state == kCTFontDescriptorMatchingDidFinishDownloading) {
        NSLog(@"字体下载完成");
    } else if (state == kCTFontDescriptorMatchingDownloading) {
        NSLog(@"下载进度 %.0f%%", progressValue);
    } else if (state == kCTFontDescriptorMatchingDidFailWithError) {
        NSError *error = [(__bridge NSDictionary *)progressParameter objectForKey:(id)kCTFontDescriptorMatchingError];
        errorDuringDownload = YES;
    }
    return YES;
});
```

通常需要在下载完字体后开始使用字体，一般是将响应代码放到 kCTFontDescriptorMatchingDidFinish 条件中，用 GCD 修改 UI 或者发 Notification 来通知相应的 Controller。
    

## 八、安全性问题

### 密文传输密码

事先生成一对用于加密的公私钥，客户端在登录时，使用公钥将用户的密码加密后，将密文传输到服务器。服务器使用私钥将密码解密，然后加盐（Salt：在密码学中，指通过在密码任意固定位置插入特定的字符串，让散列后的结果和使用原始密码的散列结果不相符）之后再多次求 MD5，之后再和服务器原来存储的用同样方法处理过的密码匹配，如果一致，则登录成功。这样，即使截获了加密后的密文，由于没有私钥，也无法还原出原始的密码。

### JSON 字段加密

如果通讯协议被破解，黑客可以模拟客户端登录，进而伪造一些用户行为，可能对用户数据造成危害。我们可以选择类似 [Protobuf](https://code.google.com/p/protobuf/protobuf)（Google 提供的开源数据交换格式，其最大特点是基于二进制，比传统的 JSON 格式要小很多）之类的二进制通讯协议或者自己实现通讯协议，对于传输的内容进行一定程度的加密。

### 程序文件加密

很多应用的部分逻辑是用 Web 方式实现的，解压包内容可以看到 js 文件，这些文件都有完整清晰的注释，很容易知道其调用逻辑。通过将 js 源码进行混淆和加密，可以防止黑客轻易地阅读和篡改相关的逻辑，也可以防止 Web 端和 Native 端的通讯协议泄漏。

### 本地数据加密

比如一款游戏，打开本地文件，可以容易地看到各种属性变量的名称和参数，我们就可以简单修改，达到修改游戏参数的目的，淘宝上许多商家就是如此操作。对于本地的重要数据，我们应该加密存储或者将其保存到 keychain 中，以保证其不被篡改。 

### 源代码安全

通过 [IDA](https://www.hex-rays.com/products/ida/) 这类工具进行反编译，可以反汇编到可以方便阅读的程度，更加方便地分析出应用的通讯协议和数据加密方式。对于这类工具，我们的应对措施就比较少了，除了用一些宏来简单混淆类名外，我们也可以将关键的逻辑用存 C 实现，例如微信的 iOS 端通讯底层，就是用 C 实现的。这样的方式除了能保证通讯协议安全外，还可以在 iOS 和 Android 使用同一套底层通讯代码，达到复用的目的。


## 九、基于 CoreText 的排版引擎

CoreText 是用于处理文字和字体的底层技术，它直接和 Core Graphics（又称为 Quartz）打交道。Quartz 是一个 2D 图形渲染引擎，能够处理 OSX 和 iOS 中的图形显示问题。使用 CoreText 技术，我们可以对富文本进行复杂的排版。经过一些简单的扩展，我们还可以实现对于图片、链接的点击效果。CoreText 技术相对于 UIWebView 有内存占用少，可以后台渲染的优点，非常适合排版工作。

### 1. CoreText 和 UIWebView 比较

![image018](/img/img018.png)

#### 优点

- CoreText 占用的内存少，渲染速度更快；
- CoreText 在渲染界面前就可以精确的获得显示内容的高度（只要有了 CTFrame 即可），而 UIWebView 只有渲染出内容后，才能获得内容的高度（而且还需要用 JavaScript 代码来获取）；
- CoreText 的 CTFrame 可以在后台线程渲染，UIWebView 的内容只能在主线程渲染；
- 基于 CoreText 可以做更好的原生交互效果，而 UIWebView 的交互效果都是用 JavaScript 来实现的，在交互效果上会有一些卡顿情况存在。

#### 缺点

- CoreText 渲染出来的内容不能像 UIWebView 那样方便的支持内容的复制；
- 基于 CoreText 来排版需要自己处理很多复杂的逻辑，例如需要自己处理图片和文字混排相关的逻辑，也需要自己实现链接点击操作的支持。

### 2. 基于 CoreText 的排版引擎框架

这部分没有细看，待总结，具体文章内容和示例点击[这里](https://github.com/Mayan29/Blog/tree/master/Notes/SDK/01.《iOS%20开发进阶（唐巧）》读书笔记)。


## 十、开发技巧

### 1. 申请加急审核

1. 访问 iTunes Connect 网站：[https://itunesconnect.apple.com](https://itunesconnect.apple.com) 
2. 单击网站底部的 Contact Us 按钮
3. 在问题 1 中选择 App Review
4. 在问题 2 中选择 Request Expedited Review
5. 单击 Request an Expedited App Review 按钮即可填写加急审核的申请表。最好使用英文填写，最容易通过的理由是严重的崩溃 bug（在理由一栏选择 Critical Bug Fix），在原因处详细描述该 bug 的重现步骤，则很容易使申请通过。

### 2. 如何将应用下架

最简单的办法是将应用的上架时间改成未来的一个时间，这样就会在数小时之内下架

### 3. 收起键盘的其他方式

在 UIViewController 中收起键盘，除了调用相应控件的 `resignFirstResponder` 方法外，还有另外三种方法：

1. 重载 UIViewController 中的 `touchesBegin` 方法，然后在里面执行 `[self.view endEditing:YES];`，这样点击 UIViewController 任意地方，就可以收起键盘；

2. 直接执行 `[[UIApplication sharedApplication] sendAction:@selector(resignFirstResponder) to:nil from:nil forEvent:nil];
`，用于在获取当前 UIViewController 比较困难的时候使用；

3. 直接执行 `[[UIApplication sharedApplication].keyWindow endEditing:YES];`

### 4. NSJSONSerialization 比 NSKeyedArchiver 更好

在选择持久化方案时，系统提供的 NSJSONSerialization 比 NSKeyedArchiver 在效率和体积上都更优。经过测试，NSJSONSerialization 比 NSKeyedArchiver 快了 7 倍，而且序列化之后的体积是 NSKeyedArchiver 的一半。

```objc
NSLog(@"NSJSONSerialization 开始存储");
    
NSMutableArray *persons1 = [NSMutableArray array];
for (int i = 0; i < 100000; i++) {
        
    [persons1 addObject:[NSJSONSerialization dataWithJSONObject:dic options:0 error:nil]];
}
    
NSLog(@"NSJSONSerialization 结束存储");
NSLog(@"NSKeyedArchiver 开始存储");

    
NSMutableArray *persons2 = [NSMutableArray array];
for (int i = 0; i < 100000; i++) {
        
    [persons2 addObject:[NSKeyedArchiver archivedDataWithRootObject:dic]];
}
    
NSLog(@"NSKeyedArchiver 结束存储");
```

测试结果为：NSJSONSerialization 用了 0.426 秒，NSKeyedArchiver 用了 4.344 秒

### 5. 设置应用内的系统控件语言

在 iOS 系统中，有时候会需要调用系统的一些 UI 控件，例如：

- 在 UIWebView 中长按会弹出系统的上下文菜单；
- 在 UIImagePickerController 中会使用系统的照相机界面；
- 在编辑状态下的 UITableViewCell 处于待删除状态时，会有一个系统的删除按钮；

以上 UI 控件中，显示的语言并不是和你当前手机的系统语言一致的，语言设置成中文，需要在 info.plist 文件中增加以下内容即可

```objc
<key>CFBundleLocalizations</key>
<array>
	<string>zh_CN</string>
	<string>en</string>
</array>
```

### 6. 用截屏功能来实现侧滑返回效果

iOS 7 以后的系统，可以通过系统提供的 API 来实现截屏功能

```objc
- (nullable UIView *)snapshotViewAfterScreenUpdates:(BOOL)afterUpdates;
```

早期的 QQ 在侧滑到一半的时候，整个当前界面被移动到了右半部分，同时在左半部分以半透明的方式露出了上一个界面。由于 ViewController 并不支持自己的 view 设置透明，所以需要我们自己实现。

为了使用截屏功能达到这种效果，我们在 NavigationController 进入到一个新的 ViewController 前，先进行截屏操作，保存当前的界面效果，然后将截到的当前界面作为参数，传递给目标 ViewController 当作背景。

这样，平时这个背景我们用内容遮挡住，当用户用手指向右滑动，我们将整个界面右移，露出这个背景，于是就会像看到了上一个 ViewController 一样。

### 7. 内存警告

1. CALayer 是一个 bitmap 图像的容器类，当 UIView 调用自身的 drawRect 时，CALayer 才会创建这个 bitmap 图像类。
2. CALayer 只占 48 Bytes，UIView 只占 96 Bytes，而一个 iPad 的全屏 UIView 的 bitmap 类会占到 12 MB 的大小！
3. 当系统发出 MemoryWarning 时，系统会自动回收 bitmap 类，但是不回收 UIView 和 CALayer 类。这样既能回收大部分内存，又能在需要 bitmap 类时，通过调用 UIView 的 drawRect 方法重建。
4. 当一段内存被分配时，它会被标记成 In use 以防止被重复使用，当内存被释放时，这段内存会被标记成 Not in use，这样在有新的内存申请时，这块内存就可能被分配给其他变量。
5. CALayer 包括的具体的 bitmap 内容的私有成员变量类型为 CABackingStore，当收到 MemoryWarning 时，CABackingStore 类型的内存区会被标记成可能再次被原变量使用。

### 8. Xcode 快捷键

#### 常用快捷键

快捷键 | 说明 
--- | ---
Cmd + Ctrl + Up | 在 .h 和 .m 文件之间切换
Cmd + Ctrl + Left | 到上 / 下一次编辑的位置
Cmd + Shift + F | 在工程中查找
Cmd + . | 结束本次调试

#### 不太常用快捷键

快捷键 | 说明 
--- | ---
Cmd + Shift + Y | 切换控制台的显示或隐藏
Cmd + 0 | 隐藏左边的导航区
Cmd + Opt + 0 | 隐藏右边的工具区
Cmd + Shift + K | 清空编译好的文件

### 9. 为工程增加 Daily Build

我们可以用 bash 写一个 Daily Build 脚本，可以参考 @lexrus 的自动打包脚本 [ios-makefile](https://github.com/lexrus/ios-makefile)


## 十一、Objective-C 对象

### 1. isa 指针

我们进入 `NSObject.h` 和 `objc.h` 可以看到，NSObject 就是一个包含 isa 指针的结构体：

![image019](/img/img019.png)

![image020](/img/img020.png)

按照面向对象语言的设计原则，所有事物都应该是对象，严格来说，Objective-C 并没有完全做到这一点，因为它有像 int、double 这样的简单变量类型，而类似 Ruby 一类语言，连 int 变量也是对象。

我们进入 `runtime.h` 中打开 Class 的定义头文件，可以看到，Class 也是一个包含 isa 指针的结构体

![image021](/img/img021.png)

类也是一个对象，所以它也必须是另一个类的实例，这个类就是元类（metaclass），元类保存了类方法的列表。当一个类方法被调用时，元类会首先查找它本身是否有该类方法的实现。

元类也是一个对象，那么元类的 isa 指针又指向哪里呢？为了设计上的完整，所有元类 isa 指针都会指向一个根元类（root metaclass），根元类本身的 isa 指针指向自己，这样就形成了一个闭环。

### 2. 类的成员变量

如果把类的实例看成一个 C 语言的结构体，上面说的 isa 指针就是这个结构体的第一个成员变量，而类的其他成员变量依次排列在结构体中。为了验证排列顺序，在 main.m 中运行如下代码：

```objc
#import <UIKit/UIKit.h>
#import "AppDelegate.h"

@interface Father : NSObject {
    int _father;
}
@end

@implementation Father
@end

@interface Child : Father {
    int _child;
}
@end

@implementation Child
@end

int main(int argc, char * argv[]) {
    
    Child *child = [[Child alloc] init];
    
    @autoreleasepool {
        return UIApplicationMain(argc, argv, nil, NSStringFromClass([AppDelegate class]));
    }
}
```

我们在 @autoreleasepool 处打断点，在 Console 中输入 `p *child` 则可以看到输出如下

![image022](/img/img022.png)

因为对象在内存中排布可以看成一个结构体，结构体的大小并不能动态变化，所以无法在运行时动态地给对象添加成员变量。但是对象的方法定义都保存在类的可变区域中，通过修改方法列表的指针，就可以动态地为某一个类增加成员变量，这也是 Category 实现的原理，同时也说明了为什么 Category 只能为对象增加成员方法，却不能增加成员变量。

### 3. 动态创建类和对象

```objc
- (void)viewDidLoad {
    [super viewDidLoad];

    // 创建一个名为 MYView 的类，它是 UIView 的子类
    Class newClass = objc_allocateClassPair([UIView class], "MYView", 0);
    
    // 为该类增加一个名为 report 的方法
    class_addMethod(newClass, NSSelectorFromString(@"report"), (IMP)ReportFunction, "v@:");
    
    // 注册该类
    objc_registerClassPair(newClass);
    
    
    // 创建一个 MYView 类的实例
    id instanceOfNewClass = [[newClass alloc] init];
    // 调用 report 方法
    [instanceOfNewClass performSelector:NSSelectorFromString(@"report")];
}

void ReportFunction(id self, SEL _cmd)
{
    NSLog(@"Class is %@, and super is %@", [self class], [self superclass]);
}
```

执行结果为：

```objc
Class is MYView, and super is UIView
```

### 4. 动态方法替换

4.1. `method_exchangeImplementations`，最常用的方法，当需要交换两个方法的实现时使用

```objc
Method viewWillAppear = class_getInstanceMethod(self, @selector(viewWillAppear:));
Method bimViewWillAppear = class_getInstanceMethod(self, @selector(bimViewWillAppear:));
method_exchangeImplementations(viewWillAppear, bimViewWillAppear);
```

4.2. `class_replaceMethod`，当被替换的方法有可能没有实现时使用

```objc
- (void)viewDidLoad {
    [super viewDidLoad];
    
    Class class = self.class;
    
    SEL originalSelector = @selector(viewWillAppear:);
    SEL swizzledSelector = @selector(bimViewWillAppear:);
    
    Method originalMethod = class_getInstanceMethod(class, originalSelector);
    Method swizzledMethod = class_getInstanceMethod(class, swizzledSelector);
    
    BOOL didAddMethod = class_addMethod(class, originalSelector, method_getImplementation(swizzledMethod), method_getTypeEncoding(swizzledMethod));
    if (didAddMethod) {
        class_replaceMethod(class, swizzledSelector, method_getImplementation(originalMethod), method_getTypeEncoding(originalMethod));
    }
}

- (void)bimViewWillAppear:(BOOL)animated {
    
    NSLog(@"调用了 bimViewWillAppear: 方法");
    
    [self bimViewWillAppear:animated];
}
```

打印结果为：

```
调用了 bimViewWillAppear: 方法
```

如果在该类中实现了 `viewWillAppear:` 方法：

```objc
- (void)viewWillAppear:(BOOL)animated {
    [super viewWillAppear:animated];

    NSLog(@"调用了 viewWillAppear: 方法");
}
```

则打印结果为：

```
调用了 viewWillAppear: 方法
```

说明 `didAddMethod` 返回 NO，没有实现方法交换。

其实使用 `viewWillAppear:` 方法并没有展示出来该方法的作用。比较实用的地方用在：子类继承父类，在子类中没有父类方法的覆写，子类的扩展类中实现父类方法交换。

4.3. `method_setImplementation`，直接为一个方法设置其实现方式时使用。

```objc
- (void)viewDidLoad {
    [super viewDidLoad];
    
    Method method1 = class_getInstanceMethod(self.class, @selector(work));
    Method method2 = class_getInstanceMethod(self.class, @selector(study));
    IMP imp2 = method_getImplementation(method2);
    
    method_setImplementation(method1, imp2);
    
    [self work];
}

- (void)work {
    NSLog(@"工作");
}

- (void)study {
    NSLog(@"学习");
}
```

打印结果为：

```objc
学习
```

其实 `method_exchangeImplementations` 方法就是调用了两次 `method_setImplementation`


## 十二、Tagged Pointer 对象

iPhone 5s 配备了首个采用 64 位架构的 A7 双核处理器，同时提出了 Tagged Pointer 的概念。对于 64 位系统，引入 Tagged Pointer 后，相关逻辑能减少一半的内存占用，3 倍的访问速度提升，100 倍的创建、销毁速度提升。

### 1. 原有系统的问题

举个例子：

- NSNumber 对象，在 32 位 CPU 下占 4 个字节，在 64 位 CPU 下占 8 个字节；
- 指针在 32 位 CPU 下占 4 个字节，在 64 位 CPU 下占 8 个字节；

所以如果没有 Tagged Pointer 对象，从 32 位机器迁移到 64 位机器中，虽然逻辑没有变化，但是对象所占用的内存会翻倍：

![image023](/img/img023.jpg)

### 2. Tagged Pointer 介绍

为了改进上面提到的内存占用和效率问题，苹果提出了 Tagged Pointer 对象。将一个对象的指针拆成两部分，一部分直接保存数据，另一部分作为特殊标记，表示这是一个特别的指针，不指向任何一个地址，64 位 CPU 下 NSNumber 的内存图变成下面这样：

![image024](/img/img024.jpg)

### 3. Tagged Pointer 特点

1. Tagged Pointer 专门用来存储小的对象，例如 NSNumber 和 NSDate；
2. Tagged Pointer 指针的值不再是地址，而是真正的值。所以实际上它不再是一个对象了，只是一个披着对象“皮”的普通变量而已，所以，它的内存并不存储在堆中；
3. 在内存读取上有着以前 3 倍的效率，创建时比以前快 106 倍。

### 4. 引用计数相关变化

在 32 位环境下，对象的引用计数都保存在一个外部的表中，每一个对象的 Retain 操作，实际包括如下 5 个步骤：

1. 获得全局的记录引用计数的 hash 表；
2. 为了线程安全，给该 hash 表加锁；
3. 查找到目标对象的引用计数值；
4. 将该引用计数值加 1，写回 hash 表；
5. 给该 hash 表解锁。

而在 64 位环境下，isa 指针也是 64 位，实际作为指针部分只用到其中 33 位，剩余的 31 位苹果使用了类似 Tagged Pointer 的概念，其中 19 位将保存对象的引用计数，这样对引用计数的操作只需要修改这个指针即可。只有当引用计数超出 19 位，才会将引用计数保存到外部表，但是这种情况是很少的。在 64 位环境下，新的 Retain 操作包括如下 5 个步骤：

1. 检查 isa 指针上面的标记位，看引用计数是否保存在 isa 变量中，如果不是，则使用以前的步骤，否则执行第 2 步；
2. 检查当前对象是否正在释放，如果是，不做任何事情；
3. 增加该对象的引用计数，但是并不马上写回到 isa 变量中；
4. 检查增加后的引用计数的值是否能够被 19 位表示，如果不是，则切换成以前的办法，否则执行第 5 步；
5. 进行一个原子的写操作，将 isa 的值写回。

虽然步骤都是 5 步，但是由于没有了全局的加锁操作，所以引用计数的更改更快了。

### 5. isa 的 bit 位含义

bit 位 | 变量名 | 意义
--- | --- | ---
1 bit | indexed | 0 表示普通的 isa，1 表示 Tagged Pointer
1 bit | has_assoc | 表示该对象是否有过 associated 对象，如果没有，在析构释放内存时可以更快
1 bit | has_cxx_dtor | 表示该对象是否有 C++ 或 ARC 的析构函数，如果没有，在析构释放内存时可以更快
30 bits | shiftcls | 类的指针
9 bits | magic | 其值固定为 0xd2，用于在调试时分辨对象是否未完成初始化
1 bit | weakly_referenced | 表示该对象是否有过 weak 对象，如果没有，在析构释放内存时可以更快 
1 bit | deallocating | 表示该对象是否正在析构
1 bit | has_sidetable_rc | 表示该对象的引用计数值是否大到无法直接在 isa 中保存
19 bits | extra_rc | 表示该对象超过 1 的引用计数值，例如，如果该对象的引用计数是 6，则 extra_rc 的值为 5 


## 十三、block 对象模型

### 1. block 内部数据结构定义

从苹果的 LLVM 项目的[开源代码](https://llvm.org/svn/llvm-project/compiler-rt/tags/Apple/Libcompiler_rt-10/BlocksRuntime/Block_private.h)中可以得到 block 数据结构定义，如下：

```objc
struct Block_descriptor {
    unsigned long int reserved;
    unsigned long int size;
    void (*copy)(void *dst, void *src);
    void (*dispose)(void *);
};

struct Block_layout {
    void *isa;
    int flags;
    int reserved; 
    void (*invoke)(void *, ...);
    struct Block_descriptor *descriptor;
    /* Imported variables. */
};
```

一个 block 实例由 6 部分构成：

1. isa 指针
2. flags，用于按 bit 位表示一些 block 的附加信息 
3. reserved，保留变量
4. invoke，函数指针，指向具体的 block 实现的函数调用地址
5. descriptor，表示该 block 的附加描述信息，主要是 size 大小，以及 copy 和 dispose 函数的指针
6. variables，capture 过来的变量，block 能够访问它外部的局部变量，就是因为将这些变量（或变量的地址）复制到了结构体中。

### 2. block 三种类型

- _NSConcreteGlobalBlock，全局的静态 block，不会访问任何外部变量；
- _NSConcreteStackBlock，保存在栈中的 block，当函数返回时会被销毁；
- _NSConcreteMallocBlock，保存在堆中的 block，当引用计数为 0 时会被销毁。

#### NSGlobalBlock

```objc
void(^myBlock)() = ^{
    NSLog(@"hello world");
};
    
NSLog(@"%@", myBlock);  // <__NSGlobalBlock__: 0x10d65b240>
```

对 NSGlobalBlock 的 retain、copy、release 操作都无效。

#### NSStackBlock 和 NSMallocBlock

在 MRC 模式下打印：

```objc
int num = 10;
void(^myBlock)() = ^{
    NSLog(@"malloc block and num = %d", num);
};
    
NSLog(@"%@", myBlock);  // <__NSStackBlock__: 0x7fff574d5a08>
```  

block 在函数退出的时候，就会被回收，如果再调用该 block 会导致 crash

在 ARC 模式下打印：

```objc
int num = 10;
void(^myBlock)() = ^{
    NSLog(@"malloc block and num = %d", num);
};
    
NSLog(@"%@", myBlock);  // <__NSMallocBlock__: 0x600000048220>
```  

这里为什么打印的是 NSMallocBlock 呢？在 ARC 模式下生成的 block 也是 NSStackBlock，只是当赋值给 strong 对象时，系统会主动对其进行 copy，将栈上复制到堆上。如果不赋值，直接打印，则为 NSStackBlock   

```objc
int num = 10;
    
NSLog(@"%@", ^{
    NSLog(@"hello world and num = %d", num);
});  // <__NSStackBlock__: 0x7fff574d5a08>
```

### 11.3 循环引用

1. 在 MRC 时代，__block 修饰避免循环引用；ARC 时代，__block 修饰同样会引起循环引用；
2. 不用 __block 修饰，block 外的变量引用，是复制其本身；使用 __block 修饰，block 外的变量引用，是复制其引用地址；
3. __block 不管是 ARC 还是 MRC 模式下都可以修饰对象和基本数据类型；__weak 只能在 ARC 模式下使用，也只能修饰对象，不能修饰基本数据类型；
4. __block 对象可以在 block 中被重新赋值，__weak 不可以。


## 后记

> 2017-04-27 一次阅读
>
> 前前后后大概半个月，利用零碎时间看完了这本书。这本书前年就有所耳闻，网上的评价也褒贬不一，唐巧写的相对来说，语言通俗易懂，感觉像是博客一样，没有什么限制，读起来很畅快。这本书适合初学者，虽然大部分内容之前都已经熟悉掌握，但是看过一遍还是很有收获的，有些知识点的总结很到位，感觉没有浪费时间。遗留的问题是 CoreText 部分内容打算结合其他资料系统的研究一下，这里就先不仔细阅读了。
> 
> 2018-01-19 二次阅读 
> 
> 大概三天时间通读了一遍，又有一点点收获吧，模糊的概念捋顺了一遍，总体来说内容很简单，用不到的功能还是没有细看，比如内购、CoreText 等等模块。