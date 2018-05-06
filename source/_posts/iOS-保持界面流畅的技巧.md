---
title: iOS 保持界面流畅的技巧
date: 2018-03-22 14:03:55
categories: iOS
tags:
---

> 本文内容整理于以下文章：
> 
> 1. [iOS 保持界面流畅的技巧 - ibireme](https://blog.ibireme.com/2015/11/12/smooth_user_interfaces_for_ios/)
> 2. [iOS 离屏渲染的研究 - 齐滇大圣](https://www.jianshu.com/p/6d24a4c29e18)
> 3. [离屏渲染优化详解：实例示范+性能测试](https://www.jianshu.com/p/ca51c9d3575b)
> 4. [优化 UITableViewCell 高度计算的那些事 - sunnyxx](http://blog.sunnyxx.com/2015/05/17/cell-height-calculation/)
> 5. [iOS 开发之多种 Cell 高度自适应实现方案的 UI 流畅度分析 - 青玉伏案](http://www.cnblogs.com/ludashi/p/5895725.html)


## 1. 屏幕显示图像的原理

首先从过去的 CRT 显示器原理说起。CRT 的电子枪按照上面方式，从上到下一行行扫描，扫描完成后显示器就呈现一帧画面，随后电子枪回到初始位置继续下一次扫描。

当电子枪换到新的一行，准备进行扫描时，显示器会发出一个__水平同步信号__（horizonal synchronization），简称 HSync；而当一帧画面绘制完成后，电子枪回复到原位，准备画下一帧前，显示器会发出一个__垂直同步信号__（vertical synchronization），简称 VSync。

显示器通常以固定频率进行刷新，这个刷新率就是__垂直同步信号__产生的频率。尽管现在的设备大都是液晶显示屏了，但原理仍然没有变。

![image001](/img/img001.png)

通常来说，计算机系统中 CPU、GPU、显示器是以上面这种方式协同工作的。CPU 计算好显示内容提交到 GPU，GPU 渲染完成后将渲染结果放入帧缓冲区（FrameBuffer），随后视频控制器会按照__垂直同步信号__逐行读取帧缓冲区的数据，经过数模转换传递给显示器显示。

在最简单的情况下，帧缓冲区只有一个，这时帧缓冲区的读取和刷新都会有比较大的效率问题。为了解决效率问题，显示系统通常会引入两个缓冲区，即双缓冲机制。在这种情况下，GPU 会预先渲染好一帧放入一个缓冲区内，让视频控制器读取，当下一帧渲染好后，GPU 会直接把视频控制器的指针指向第二个缓冲器。如此一来效率会有很大的提升。

双缓冲虽然能解决效率问题，但会引入一个新的问题。当视频控制器还未读取完成时，即屏幕内容刚显示一半时，GPU 将新的一帧内容提交到帧缓冲区并把两个缓冲区进行交换后，视频控制器就会把新的一帧数据的下半段显示到屏幕上，造成画面撕裂现象，如下图：

![image002](/img/img002.jpg)

为了解决这个问题，GPU 通常有一个机制叫做__垂直同步__（简写也是 V-Sync），当开启垂直同步后，GPU 会等待显示器的__垂直同步信号__发出后，才进行新的一帧渲染和缓冲区更新。这样能解决画面撕裂现象，也增加了画面流畅度，但需要消费更多的计算资源，也会带来部分延迟。

iOS 设备会始终使用双缓存，并开启垂直同步。而安卓设备直到 4.1 版本，Google 才开始引入这种机制，目前安卓系统是三缓存+垂直同步。


## 2. 卡顿产生的原因

![image003](/img/img003.png)

在__垂直同步信号__到来后，系统图形服务会通过 CADisplayLink 等机制通知 App，App 主线程开始在 CPU 中计算显示内容，比如视图的创建、布局计算、图片解码、文本绘制等。随后 CPU 会将计算好的内容提交到 GPU 去，由 GPU 进行变换、合成、渲染。随后 GPU 会把渲染结果提交到帧缓冲区去，等待下一次__垂直同步信号__到来时显示到屏幕上。由于垂直同步的机制，如果在一个__垂直同步信号__时间内，CPU 或者 GPU 没有完成内容提交，则那一帧就会被丢弃，等待下一次机会再显示，而这时显示屏会保留之前的内容不变。这就是界面卡顿的原因。

从上图中可以看到，CPU 和 GPU 不论哪个阻碍了显示流程，都会造成掉帧现象。所以开发时，也需要分别对 CPU 和 GPU 压力进行评估和优化。

> 结合上图，这里我个人的理解是：
> 
> 假设双缓存机制中两个帧缓冲区命名为：FrameBufferA 和 FrameBufferB。第一个__垂直同步信号__发出后，视频控制器读取的是 FrameBufferA 中准备好的数据以供显示，CPU 和 GPU 所计算和渲染的是下一帧准备放入 FrameBufferB 中的数据。以此类推，第二个__垂直同步信号__发出后，视频控制器读取的是 FrameBufferB 中的数据。。。
> 
> 希望 iOS 早日能像安卓系统一样使用三缓存机制 ^_^


## 3. CPU 资源消耗原因和解决方案

### 对象创建

对象的创建会分配内存、调整属性、甚至还有读取文件等操作，比较消耗 CPU 资源。

1. 尽量用轻量的对象代替重量的对象：比如 CALayer 比 UIView 要轻量许多，那么不需要响应触摸事件的控件，用 CALayer 显示会更加合适。
2. 如果对象不涉及 UI 操作，则尽量放到后台线程去创建，但可惜的是包含有 CALayer 的控件，都只能在主线程创建和操作。
3. 通过 Storyboard 创建视图对象时，其资源消耗会比直接通过代码创建对象要大非常多，在性能敏感的界面里不要使用 Storyboard。
4. 如果对象可以复用，并且复用的代价比释放、创建新对象要小，那么这类对象应当尽量放到一个缓存池里复用。

### 对象调整

对象的调整也经常是消耗 CPU 资源的地方，这里特别说一下 CALayer：CALayer 内部并没有属性，当调用属性方法时，它内部是通过 runtime 为对象临时添加一个方法，并把对应属性值保存到内部的一个 Dictionary 里，同时还会通知 delegate、创建动画等等，非常消耗资源。UIView 的关于显示相关的属性（比如 frame/bounds/transform）等实际上都是 CALayer 属性映射来的，所以对这些属性进行调整时，消耗的资源要远大于一般的属性。对此你在应用中，应该尽量减少不必要的属性修改。

### 对象销毁

对象的销毁虽然消耗资源不多，但累积起来也是不容忽视的。同样的，如果对象可以放到后台线程去释放，那就挪到后台线程去。这里有个小 Tip：把对象捕获到 block 中，然后扔到后台队列去随便发送个消息以避免编译器警告，就可以让对象在后台线程销毁了。

```objc
NSArray *tmp = self.array;
self.array = nil;
dispatch_async(queue, ^{
    [tmp class];
});
```

### Autolayout

Autolayout 是苹果本身提倡的技术，在大部分情况下也能很好的提升开发效率，但是 Autolayout 对于复杂视图来说会产生严重的性能问题。随着视图数量的增长，Autolayout 带来的 CPU 消耗会呈指数级上升。具体数据可以看这个文章：[http://pilky.me/36/](http://pilky.me/36/)。

### 文本计算

如果一个界面中包含大量文本（比如微博微信朋友圈等），文本的宽高计算会占用很大一部分资源，并且不可避免。如果你对文本显示没有特殊要求，可以参考下 UILabel 内部的实现方式：用 `[string boundingRectWithSize:options:context:]` 来计算文本宽高，用 `[string drawWithRect:options:attributes:context:]` 来绘制文本。尽管这两个方法性能不错，但仍旧需要放到后台线程进行以避免阻塞主线程。

如果你用 CoreText 绘制文本，那就可以先生成 CoreText 排版对象，然后自己计算了，并且 CoreText 对象还能保留以供稍后绘制使用。

### 文本渲染

屏幕上能看到的所有文本内容控件，包括 UIWebView，在底层都是通过 CoreText 排版、绘制为 Bitmap 显示的。常见的文本控件 （UILabel、UITextView 等），其排版和绘制都是在主线程进行的，当显示大量文本时，CPU 的压力会非常大。对此解决方案只有一个，那就是自定义文本控件，用底层的 CoreText 对文本异步绘制。尽管这实现起来非常麻烦，但其带来的优势也非常大，CoreText 对象创建好后，能直接获取文本的宽高等信息，避免了多次计算（调整 UILabel 大小时算一遍、UILabel 绘制时内部再算一遍）。而且 CoreText 对象占用内存较少，可以缓存下来以备稍后多次渲染。

### 图片的解码

当你用 UIImage 或 CGImageSource 的那几个方法创建图片时，图片数据并不会立刻解码。图片设置到 UIImageView 或者 CALayer.contents 中去，并且 CALayer 被提交到 GPU 前，CGImage 中的数据才会得到解码。这一步是发生在主线程的，并且不可避免。如果想要绕开这个机制，常见的做法是在后台线程先把图片绘制到 CGBitmapContext 中，然后从 Bitmap 直接创建图片。目前常见的网络图片库都自带这个功能。

### 图像的绘制

图像的绘制通常是指用那些以 CG 开头的方法把图像绘制到画布中，然后从画布创建图片并显示这样一个过程。这个最常见的地方就是 `[UIView drawRect:]` 里面了。由于 CoreGraphic 方法通常都是线程安全的，所以图像的绘制可以很容易的放到后台线程进行。一个简单异步绘制的过程大致如下（实际情况会比这个复杂得多，但原理基本一致）：

```objc
- (void)display {
    dispatch_async(backgroundQueue, ^{
        CGContextRef ctx = CGBitmapContextCreate(...);
        // draw in context...
        CGImageRef img = CGBitmapContextCreateImage(ctx);
        CFRelease(ctx);
        dispatch_async(mainQueue, ^{
            layer.contents = img;
        });
    });
}
```


## 4. GPU 资源消耗原因和解决方案

相对于 CPU 来说，GPU 能干的事情比较单一：接收提交的纹理（Texture）和顶点描述（三角形），应用变换（transform）、混合并渲染，然后输出到屏幕上。通常你所能看到的内容，主要也就是纹理（图片）和形状（三角模拟的矢量图形）两类。

### 纹理的渲染（图片和视图）

所有的 Bitmap，包括图片、文本、栅格化的内容，最终都要由内存提交到显存，绑定为 GPU Texture（是一个对 GPU 只读的资源，其有着一些属性，宽度、高度、色彩通道数量等）。不论是提交到显存的过程，还是 GPU 调整和渲染 Texture 的过程，都要消耗不少 GPU 资源。当在较短时间显示大量图片时（比如 TableView 存在非常多的图片并且快速滑动时），CPU 占用率很低，GPU 占用非常高，界面仍然会掉帧。避免这种情况的方法只能是尽量减少在短时间内大量图片的显示，尽可能将多张图片合成为一张进行显示。

当图片过大，超过 GPU 的最大纹理尺寸时，图片需要先由 CPU 进行预处理，这对 CPU 和 GPU 都会带来额外的资源消耗。目前来说，iPhone 4S 以上机型，纹理尺寸上限都是 4096×4096，更详细的资料可以看这里：[iosres.com](http://www.iosres.com)。所以，尽量不要让图片和视图的大小超过这个值。

### 视图的混合（opaque 属性） 

当多个视图（或者说 CALayer）重叠在一起显示时，GPU 会首先把他们混合到一起。如果视图结构过于复杂，混合的过程也会消耗很多 GPU 资源。为了减轻这种情况的 GPU 消耗，应用应当尽量减少视图数量和层次，并在不透明的视图里标明 opaque 属性 YES 以避免无用的 Alpha 通道合成。当然，这也可以用上面的方法，把多个视图预先渲染为一张图片来显示。

> opaque 属性表示当前控件是否不透明，UIView 默认值是 YES，但 UIButton 等子类的默认值都是 NO。为了提升性能，如果开发中 UIView 是不透明的，opaque 设置为 YES，如果  Alpha 小于 1，则 opaque 应该设置为 NO。

### 图形的生成（离屏渲染）

#### a. 当前屏幕渲染与离屏渲染

OpenGL 中 GPU 屏幕渲染有两种方式：当前屏幕渲染（GPU 的渲染操作是在当前用于显示的屏幕缓冲区中进行）和离屏渲染（GPU 在当前屏幕缓冲区以外新开辟一个缓冲区进行渲染操作）。

相比于当前屏幕渲染，离屏渲染的代价是很高的，主要体现在两个方面：创建新缓冲区，上下文切换。

要想进行离屏渲染，首先要创建一个新的缓冲区。离屏渲染的整个过程，需要多次切换上下文环境：先是从当前屏幕（On-Screen）切换到离屏（Off-Screen）；等到离屏渲染结束以后，将离屏缓冲区的渲染结果显示到屏幕上有需要将上下文环境从离屏切换到当前屏幕。而上下文环境的切换是很耗性能的。

#### b. 离屏渲染触发方式

- shouldRasterize（光栅化）
- masks（遮罩）
- shadows（阴影）
- edge antialiasing（抗锯齿）
- group opacity（不透明）

其中 shouldRasterize（光栅化）是比较特别的一种，如果 `CALayer.shouldRasterize =  YES`，在其他属性触发离屏绘制的同时，会将光栅化后的内容缓存起来，如果对应的 layer 及其 sublayers 没有发生改变，在下一帧的时候可以直接复用。这样会隐式的创建一个位图，各种阴影遮罩等效果也会保存到位图中并缓存起来，相当于把 GPU 的操作转到 CPU 上了，生成位图缓存，直接读取复用，这将在很大程度上提升渲染性能。但是对于经常变动的内容，建议不要开启，否则会造成性能的浪费。比如 TableViewCell 重绘是很频繁的，因为复用，cell 需要不断的重绘，如果此时设置了 cell.layer 可光栅化，则会造成大量的离屏渲染，降低图形性能。

> 光栅化：将图转化成一个个栅格组成的图像，每个元素对应帧缓冲区中的一个像素。

对于只需要圆角的某些场合，也可以用一张已经绘制好的圆角图片覆盖到原本视图上面来模拟相同的视觉效果。最彻底的解决办法，就是把需要显示的图形在后台线程绘制为图片，避免使用圆角、阴影、遮罩等属性。

#### c. 另一种特殊的离屏渲染：CPU 渲染

如果我们重写了 drawRect 方法，并且使用任何 Core Graphics 的技术进行了绘制操作，就涉及到了 CPU 渲染。整个渲染过程由 CPU 在 App 内同步地完成，渲染得到的 bitmap 最后再交由 GPU 用于显示。

由于 GPU 的浮点运算能力比 CPU 强，CPU 渲染的效率可能不如离屏渲染；但如果仅仅是实现一个简单的效果，直接使用 CPU 渲染的效率又可能比离屏渲染好，毕竟离屏渲染要涉及到缓冲区创建和上下文切换等耗时操作。

#### d. 离屏渲染检测工具：Instruments

Instruments 的 Core Animation 工具中有两个和离屏渲染相关的检查选项：

1. __Color__ __Offscreen-Rendered__ __Yellow__
开启后会把那些需要离屏渲染的图层高亮成黄色，这就意味着黄色图层可能存在性能问题。

2. __Color__ __Hits__ __Green__ __and__ __Misses__ __Red__
这一项是检查该场景下光栅化操作是否是一个好的选择，如果图层是绿色，就表示这些缓存被复用；如果是红色就表示缓存会被重复创建，这就表示该处存在性能问题了。

#### e. iOS 版本上的优化

iOS 9.0 之前 UIimageView 跟 UIButton 设置圆角都会触发离屏渲染

iOS 9.0 之后 UIButton 设置圆角会触发离屏渲染，而 UIImageView 里 png 图片设置圆角不会触发离屏渲染了，如果设置其他阴影效果之类的还是会触发离屏渲染的。

这可能是苹果也意识到离屏渲染会产生性能问题，所以能不产生离屏渲染的地方苹果也就不用离屏渲染了。


## 5. AsyncDisplayKit 思维方式

AsyncDisplayKit 是 Facebook 开源的一个用于保持 iOS 界面流畅的库，其作者是 Scott Goodson ([Linkedin](https://www.linkedin.com/in/iosengineer))

### ASDK 基本原理

![image004](/img/img004.png)

ASDK 认为，阻塞主线程的任务，主要分为上面这三大类。文本和布局的计算、渲染、解码、绘制都可以通过各种方式异步执行，但 UIKit 和 Core Animation 相关操作必需在主线程进行。ASDK 的目标，就是尽量把这些任务从主线程挪走，而挪不走的，就尽量优化性能。

常见的 UIView 和 CALayer 的关系：View 持有 Layer 用于显示，View 中大部分显示属性实际是从 Layer 映射而来；Layer 的 delegate 在这里是 View，当其属性改变、动画产生时，View 能够得到通知。UIView 和 CALayer 不是线程安全的，并且只能在主线程创建、访问和销毁。

![image005](/img/img005.png)

ASDK 为此创建了 ASDisplayNode 类，包装了常见的视图属性（比如 frame/bounds/alpha/transform/backgroundColor/superNode/subNodes 等），然后它用 UIView -> CALayer 相同的方式，实现了 ASNode -> UIView 这样一个关系。

![image006](/img/img006.png)

当不需要响应触摸事件时，ASDisplayNode 可以被设置为 layer backed，即 ASDisplayNode 充当了原来 UIView 的功能，节省了更多资源。

与 UIView 和 CALayer 不同，ASDisplayNode 是线程安全的，它可以在后台线程创建和修改。Node 刚创建时，并不会在内部新建 UIView 和 CALayer，直到第一次在主线程访问 view 或 layer 属性时，它才会在内部生成对应的对象。当它的属性（比如frame/transform）改变后，它并不会立刻同步到其持有的 view 或 layer 去，而是把被改变的属性保存到内部的一个中间变量，稍后在需要时，再通过某个机制一次性设置到内部的 view 或 layer。

通过模拟和封装 UIView/CALayer，开发者可以把代码中的 UIView 替换为 ASNode，很大的降低了开发和学习成本，同时能获得 ASDK 底层大量的性能优化。为了方便使用，ASDK 把大量常用控件都封装成了 ASNode 的子类，比如 Button、Control、Cell、Image、ImageView、Text、TableView、CollectionView 等。利用这些控件，开发者可以尽量避免直接使用 UIKit 相关控件，以获得更完整的性能提升。

### ASDK 的图层预合成

![image007](/img/img007.png)![image008](/img/img008.png)

有时一个 layer 会包含很多 sub-layer，而这些 sub-layer 并不需要响应触摸事件，也不需要进行动画和位置调整。ASDK 为此实现了一个被称为 pre-composing 的技术，可以把这些 sub-layer 合成渲染为一张图片。开发时，ASNode 已经替代了 UIView 和 CALayer；直接使用各种 Node 控件并设置为 layer backed 后，ASNode 甚至可以通过预合成来避免创建内部的 UIView 和 CALayer。

通过这种方式，把一个大的层级，通过一个大的绘制方法绘制到一张图上，性能会获得很大提升。CPU 避免了创建 UIKit 对象的资源消耗，GPU 避免了多张 texture 合成和渲染的消耗，更少的 bitmap 也意味着更少的内存占用。

### ASDK 异步并发操作

充分利用多核的优势、并发执行任务对保持界面流畅有很大作用。ASDK 把布局计算、文本排版、图片/文本/图形渲染等操作都封装成较小的任务，并利用 GCD 异步并发执行。如果开发者使用了 ASNode 相关的控件，那么这些并发操作会自动在后台进行，无需进行过多配置。

### Runloop 任务分发

![image009](/img/img009.png)

Runloop work distribution 是 ASDK 比较核心的一个技术，iOS 的显示系统是由__垂直同步信号__驱动的，__垂直同步信号__由硬件时钟生成，每秒钟发出 60 次（这个值取决设备硬件，比如 iPhone 真机上通常是 59.97）。iOS 图形服务接收到__垂直同步信号__后，会通过 IPC 通知到 App 内。App 的 Runloop 在启动后会注册对应的 CFRunLoopSource 接收传过来的时钟信号通知，随后 Source 的回调会驱动整个 App 的动画与显示。

Core Animation 在 RunLoop 中注册了一个 Observer，监听了 BeforeWaiting 和 Exit 事件。当一个触摸事件到来时，RunLoop 被唤醒，App 中的代码会执行一些操作，比如创建和调整视图层级、设置 UIView 的 frame、修改 CALayer 的透明度、为视图添加一个动画；这些操作最终都会被 CALayer 捕获，并通过 CATransaction 提交到一个中间状态去。当上面所有操作结束后，RunLoop 即将进入休眠（或者退出）时，关注该事件的 Observer 都会得到通知。这时 Core Animation 注册的那个 Observer 就会在回调中，把所有的中间状态合并提交到 GPU 去显示；如果此处有动画，CA 会通过 DisplayLink 等机制多次触发相关流程。

ASDK 在此处模拟了 Core Animation 的这个机制：所有针对 ASNode 的修改和提交，总有些任务是必需放入主线程执行的。当出现这种任务时，ASNode 会把任务用 ASAsyncTransaction(Group) 封装并提交到一个全局的容器去。ASDK 也在 RunLoop 中注册了一个 Observer，监视的事件和 Core Animation 一样，但优先级要低。当 RunLoop 进入休眠前、CA 处理完事件后，ASDK 就会执行该 loop 内提交的所有任务。

通过这种机制，ASDK 可以在合适的机会把异步、并发的操作同步到主线程去，并且能获得不错的性能。


## 6. 微博性能优化技巧

### 预排版

当获取到 JSON 数据后，我会把每条 Cell 需要的数据都在后台线程计算并封装为一个布局对象 CellLayout。CellLayout 包含所有文本的 CoreText 排版结果、Cell 内部每个控件的高度、Cell 的整体高度。每个 CellLayout 的内存占用并不多，所以当生成后，可以全部缓存到内存，以供稍后使用。这样，TableView 在请求各个高度函数时，不会消耗任何多余计算量；当把 CellLayout 设置到 Cell 内部时，Cell 内部也不用再计算布局了。

如果你对性能的要求并不那么高，可以尝试用 TableView 的预估高度的功能，并把每个 Cell 高度缓存下来。这里有个来自百度知道团队的开源项目可以很方便的帮你实现这一点：[FDTemplateLayoutCell](https://github.com/forkingdog/UITableView-FDTemplateLayoutCell/)。

### 预渲染

微博的头像在某次改版中换成了圆形，当头像下载下来后，我会在后台线程将头像预先渲染为圆形并单独保存到一个 ImageCache 中去。

对于 TableView 来说，Cell 内容的离屏渲染会带来较大的 GPU 消耗。如果用 layer 的圆角属性，查看 Instument 时能够看到 GPU 已经满负荷运转，而 CPU 却比较清闲。为了避免离屏渲染，你应当尽量避免使用 layer 的 border、corner、shadow、mask 等技术，而尽量在后台线程预先绘制好对应内容。

### 异步绘制

我只在显示文本的控件上用到了异步绘制的功能，但效果很不错。我参考 ASDK 的原理，实现了一个简单的异步绘制控件。这块代码我单独提取出来，放到了这里：[YYAsyncLayer](https://github.com/ibireme/YYAsyncLayer)。YYAsyncLayer 是 CALayer 的子类，当它需要显示内容（比如调用了 `[layer setNeedDisplay]`）时，它会向 delegate，也就是 UIView 请求一个异步绘制的任务。在异步绘制时，Layer 会传递一个 `BOOL(^isCancelled)()` 这样的 block，绘制代码可以随时调用该 block 判断绘制任务是否已经被取消。

当 TableView 快速滑动时，会有大量异步绘制任务提交到后台线程去执行。但是有时滑动速度过快时，绘制任务还没有完成就可能已经被取消了。如果这时仍然继续绘制，就会造成大量的 CPU 资源浪费，甚至阻塞线程并造成后续的绘制任务迟迟无法完成。我的做法是尽量快速、提前判断当前绘制任务是否已经被取消；在绘制每一行文本前，我都会调用 isCancelled() 来进行判断，保证被取消的任务能及时退出，不至于影响后续操作。

目前有些第三方微博客户端（比如 VVebo、墨客等），使用了一种方式来避免高速滑动时 Cell 的绘制过程，相关实现见这个项目：[VVeboTableViewDemo](https://github.com/johnil/VVeboTableViewDemo)。它的原理是，当滑动时，松开手指后，立刻计算出滑动停止时 Cell 的位置，并预先绘制那个位置附近的几个 Cell，而忽略当前滑动中的 Cell。这个方法比较有技巧性，并且对于滑动性能来说提升也很大，唯一的缺点就是快速滑动中会出现大量空白内容。如果你不想实现比较麻烦的异步绘制但又想保证滑动的流畅性，这个技巧是个不错的选择。

### 全局并发控制

当用__并发队列__来执行大量绘制任务时，偶尔会遇到这种问题：大量的任务提交到后台队列时，某些任务会因为某些原因（此处是 CGFont 锁）被锁住导致线程休眠，或者被阻塞，__并发队列__随后会创建新的线程来执行其他任务。当这种情况变多时，或者 App 中使用了大量__并发队列__来执行较多任务时，App 在同一时刻就会存在几十个线程同时运行、创建、销毁。CPU 是用时间片轮转来实现线程并发的，尽管__并发队列__能控制线程的优先级，但当大量线程同时创建运行销毁时，这些操作仍然会挤占掉主线程的 CPU 资源。ASDK 有个 Feed 列表的 Demo：[SocialAppLayout](https://github.com/facebookarchive/AsyncDisplayKit/tree/master/examples/SocialAppLayout)，当列表内 Cell 过多，并且非常快速的滑动时，界面仍然会出现少量卡顿，我谨慎的猜测可能与这个问题有关。

使用__并发队列__时不可避免会遇到这种问题，但使用__串行队列__又不能充分利用多核 CPU 的资源。我写了一个简单的工具 [YYDispatchQueuePool](https://github.com/ibireme/YYDispatchQueuePool)，为不同优先级创建和 CPU 数量相同的 __串行队列__，每次从 pool 中获取 queue 时，会轮询返回其中一个 queue。我把 App 内所有异步操作，包括图像解码、对象释放、异步绘制等，都按优先级不同放入了全局的__并发队列__中执行，这样尽量避免了过多线程导致的性能问题。

### 更高效的异步图片加载

SDWebImage 在这个 Demo 里仍然会产生少量性能问题，并且有些地方不能满足我的需求，所以我自己实现了一个性能更高的图片加载库。在显示简单的单张图片时，利用 UIView.layer.contents 就足够了，没必要使用 UIImageView 带来额外的资源消耗，为此我在 CALayer 上添加了 setImageWithURL 等方法。除此之外，我还把图片解码等操作通过 YYDispatchQueuePool 进行管理，控制了 App 总线程数量。

### 其他可以改进的地方

上面这些优化做完后，微博 Demo 已经非常流畅了，但在我的设想中，仍然有一些进一步优化的技巧，但限于时间和精力我并没有实现，下面简单列一下：

1. 列表中有不少视觉元素并不需要触摸事件，这些元素可以用 ASDK 的图层合成技术预先绘制为一张图。

2. 再进一步减少每个 Cell 内图层的数量，用 CALayer 替换掉 UIView。

3. 目前每个 Cell 的类型都是相同的，但显示的内容却各部一样，比如有的 Cell 有图片，有的 Cell 里是卡片。把 Cell 按类型划分，进一步减少 Cell 内不必要的视图对象和操作，应该能有一些效果。

4. 把需要放到主线程执行的任务划分为足够小的块，并通过 Runloop 来进行调度，在每个 Loop 里判断下一次 VSync 的时间，并在下次 VSync 到来前，把当前未执行完的任务延迟到下一个机会去。这个只是我的一个设想，并不一定能实现或起作用。


## 7. 如何评测界面的流畅度

最后还是要提一下，“过早的优化是万恶之源”，在需求未定，性能问题不明显时，没必要尝试做优化，而要尽量正确的实现功能。

如果你需要一个明确的 FPS 指示器，可以尝试一下 [KMCGeigerCounter](https://github.com/kconner/KMCGeigerCounter)。对于 CPU 的卡顿，它可以通过内置的 CADisplayLink 检测出来；对于 GPU 带来的卡顿，它用了一个 1×1 的 SKView 来进行监视。这个项目有两个小问题：SKView 虽然能监视到 GPU 的卡顿，但引入 SKView 本身就会对 CPU/GPU 带来额外的一点的资源消耗；这个项目在 iOS 9 下有一些兼容问题，需要稍作调整。

我自己也写了个简单的 FPS 指示器：[FPSLabel](https://github.com/ibireme/YYText/blob/master/Demo/YYTextDemo/YYFPSLabel.m) 只有几十行代码，仅用到了 CADisplayLink 来监视 CPU 的卡顿问题。虽然不如上面这个工具完善，但日常使用没有太大问题。

最后，用 Instuments 的 GPU Driver 预设，能够实时查看到 CPU 和 GPU 的资源消耗。在这个预设内，你能查看到几乎所有与显示有关的数据，比如 Texture 数量、CA 提交的频率、GPU 消耗等，在定位界面卡顿的问题时，这是最好的工具。
