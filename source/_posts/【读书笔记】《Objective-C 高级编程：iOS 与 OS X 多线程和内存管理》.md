---
title: 【读书笔记】《Objective-C 高级编程：iOS 与 OS X 多线程和内存管理》
date: 2018-02-06 10:31:22
categories: 读书笔记
tags:
---


> 作者：坂本一树 / 古本智彦 
> 
> 出版日期：2013-6-1
> 
> 豆瓣地址：https://book.douban.com/subject/24720270/


## 1. 引用计数内部实现

查看引用计数相关源代码，可以看出每次都会取得 `CFBasicHashRef` 进行操作，由此可知苹果采用散列表（引用计数表）来管理引用计数。

> 散列表（Hash table，也叫哈希表），是根据 key 而直接访问在内存存储位置的数据结构。也就是说，它通过计算一个关于键值的函数，将所需查询的数据映射到表中一个位置来访问记录，这就加快了查找速度。这个映射函数称作散列函数，存放记录的数组称作散列表。
> 
> 一个通俗的例子是，为了查找电话簿中某人的号码，可以创建一个按照人名首字母顺序排列的表（即建立人名 x 到首字母 F(x) 的一个函数关系），在首字母为 W 的表中查找`王`姓的电话号码，显然比直接查找就要快得多。这里使用人名作为关键字，`取首字母`是这个例子中散列函数的函数法则 F()，存放首字母的表对应散列表。关键字和函数法则理论上可以任意确定。

引用计数表各记录中存有内存块地址，可以各个记录追溯到各对象的内存块。这一特性在调试时有着重要的作用：即使出现故障导致对象占用的内存块损坏，但只要引用计数表没有被破坏，就能够确认各内存块的位置：

![image029](/img/img029.png)

另外，在利用工具检测内存泄漏时，引用计数表的各记录也有助于检测各对象的持有者是否存在。


## 2. Block 的本质

### 例 1

```objc
int main(int argc, const char * argv[]) {
    
    void (^blk)(void) = ^{
        printf("Hello World.");
    };
    blk();
    return 0;
}
```

使用终端 `$ clang -rewrite-objc main.m` 将 Objective-C 代码转化成 C++ 源代码：

```c++
struct __block_impl {
  void *isa;
  int Flags;
  int Reserved;
  void *FuncPtr;
};

struct __main_block_impl_0 {
  struct __block_impl impl;
  struct __main_block_desc_0* Desc;

  __main_block_impl_0(void *fp, struct __main_block_desc_0 *desc, int flags=0) {
    impl.isa = &_NSConcreteStackBlock;
    impl.Flags = flags;
    impl.FuncPtr = fp;
    Desc = desc;
  }
};

static void __main_block_func_0(struct __main_block_impl_0 *__cself) {
  printf("Hello World.");
}

static struct __main_block_desc_0 {
  size_t reserved;
  size_t Block_size;
} __main_block_desc_0_DATA = { 
  0, 
  sizeof(struct __main_block_impl_0)
};

int main(int argc, const char * argv[]) {

    void (*blk)(void) = 
      ((void (*)())&__main_block_impl_0(
        (void *)__main_block_func_0, &__main_block_desc_0_DATA)
      );

    ((void (*)(__block_impl *))((__block_impl *)blk)->FuncPtr)((__block_impl *)blk);
    
    return 0;
}
```

我们来看 `main` 中构造函数的调用，因为转换较多，看起来不是很清楚，所以我们去掉转换的部分，具体如下：

```c++
int main(int argc, const char * argv[]) {

    struct __main_block_impl_0 tmp = __main_block_impl_0(__main_block_func_0, &__main_block_desc_0_DATA);
    struct __main_block_impl_0 *blk = &tmp;
    
    return 0;
}
```

这样就容易理解了，该源代码中的 block 就是 `__main_block_impl_0` 结构体类型的局部变量，即栈上生成的 `__main_block_impl_0` 结构体实例。再来看 block 调用部分，这就是简单地使用函数指针调用函数。由 block 语法转换的 `__main_block_func_0` 函数指针被赋值成员变量 `FuncPtr` 中，在调用该函数的源代码中可以看出 block 正是作为参数进行了传递。所以 block 的本质即为 Objective-C 的对象。

### 例 2

```objc
int main(int argc, const char * argv[]) {
    
    int num1 = 10;
    int num2 = 20;
    void (^blk)(void) = ^{
        printf("%d, %d", num1, num2);
    };
    blk();
    return 0;
}
```

使用终端 `$ clang -rewrite-objc main.m` 将 Objective-C 代码转化成 C++ 源代码：

```c++
struct __block_impl {
  void *isa;
  int Flags;
  int Reserved;
  void *FuncPtr;
};

struct __main_block_impl_0 {
  struct __block_impl impl;
  struct __main_block_desc_0* Desc;

  int num1;
  int num2;

  __main_block_impl_0(void *fp, struct __main_block_desc_0 *desc, int _num1, int _num2, int flags=0) : num1(_num1), num2(_num2) {
    impl.isa = &_NSConcreteStackBlock;
    impl.Flags = flags;
    impl.FuncPtr = fp;
    Desc = desc;
  }
};

static void __main_block_func_0(struct __main_block_impl_0 *__cself) {
  int num1 = __cself->num1; // bound by copy
  int num2 = __cself->num2; // bound by copy

  printf("%d, %d", num1, num2);
}

static struct __main_block_desc_0 {
  size_t reserved;
  size_t Block_size;
} __main_block_desc_0_DATA = {
  0, 
  sizeof(struct __main_block_impl_0)
};

int main(int argc, const char * argv[]) {

    int num1 = 10;
    int num2 = 20;
    void (*blk)(void) = 
      ((void (*)())&__main_block_impl_0(
        (void *)__main_block_func_0, &__main_block_desc_0_DATA, num1, num2)
      );

    ((void (*)(__block_impl *))((__block_impl *)blk)->FuncPtr)((__block_impl *)blk);

    return 0;
}
```

与上一个例子相比，block 语法表达式中使用的局部变量被作为成员变量追加到了 `__main_block_impl_0` 结构体中。总的来说，所谓截取局部变量值意味着在执行 block 语法时，block 语法表达式所使用的自动变量值被保存到 block 自身中。

### 例 3

```objc
int main(int argc, const char * argv[]) {
    
    __block int num1 = 10;
    __block int num2 = 20;
    void (^blk)(void) = ^{
        num1 = 100;
        num2 = 200;
        printf("%d, %d", num1, num2);
    };
    blk();
    return 0;
}
```

使用终端 `$ clang -rewrite-objc main.m` 将 Objective-C 代码转化成 C++ 源代码：

```c++
struct __block_impl {
  void *isa;
  int Flags;
  int Reserved;
  void *FuncPtr;
};

struct __Block_byref_num1_0 {
  void *__isa;
  __Block_byref_num1_0 *__forwarding;
  int __flags;
  int __size;
  int num1;
};
struct __Block_byref_num2_1 {
  void *__isa;
  __Block_byref_num2_1 *__forwarding;
  int __flags;
  int __size;
  int num2;
};

struct __main_block_impl_0 {
  struct __block_impl impl;
  struct __main_block_desc_0* Desc;

  __Block_byref_num1_0 *num1; // by ref
  __Block_byref_num2_1 *num2; // by ref

  __main_block_impl_0(void *fp, struct __main_block_desc_0 *desc, __Block_byref_num1_0 *_num1, __Block_byref_num2_1 *_num2, int flags=0) : num1(_num1->__forwarding), num2(_num2->__forwarding) {
    impl.isa = &_NSConcreteStackBlock;
    impl.Flags = flags;
    impl.FuncPtr = fp;
    Desc = desc;
  }
};

static void __main_block_func_0(struct __main_block_impl_0 *__cself) {
  __Block_byref_num1_0 *num1 = __cself->num1; // bound by ref
  __Block_byref_num2_1 *num2 = __cself->num2; // bound by ref

  (num1->__forwarding->num1) = 100;
  (num2->__forwarding->num2) = 200;
  printf("%d, %d", (num1->__forwarding->num1), (num2->__forwarding->num2));
}

static void __main_block_copy_0(struct __main_block_impl_0*dst, struct __main_block_impl_0*src) {
    _Block_object_assign(
      (void*)&dst->num1, 
      (void*)src->num1, 
      8/*BLOCK_FIELD_IS_BYREF*/);
    _Block_object_assign(
      (void*)&dst->num2, 
      (void*)src->num2,
       8/*BLOCK_FIELD_IS_BYREF*/);
  }

static void __main_block_dispose_0(struct __main_block_impl_0*src) {
  _Block_object_dispose(
    (void*)src->num1,
    8/*BLOCK_FIELD_IS_BYREF*/);
  _Block_object_dispose(
    (void*)src->num2, 
    8/*BLOCK_FIELD_IS_BYREF*/);
}

static struct __main_block_desc_0 {
  size_t reserved;
  size_t Block_size;
  void (*copy)(struct __main_block_impl_0*, struct __main_block_impl_0*);
  void (*dispose)(struct __main_block_impl_0*);
} __main_block_desc_0_DATA = { 
  0, 
  sizeof(struct __main_block_impl_0), __main_block_copy_0, __main_block_dispose_0
};

int main(int argc, const char * argv[]) {

    __attribute__((__blocks__(byref))) __Block_byref_num1_0 num1 = {
      (void*)0,
      (__Block_byref_num1_0 *)&num1, 
      0, 
      sizeof(__Block_byref_num1_0), 
      10
    };
    __attribute__((__blocks__(byref))) __Block_byref_num2_1 num2 = {
      (void*)0,
      (__Block_byref_num2_1 *)&num2,
       0, 
       sizeof(__Block_byref_num2_1), 
       20
    };
    void (*blk)(void) = ((void (*)())&__main_block_impl_0((void *)__main_block_func_0, &__main_block_desc_0_DATA, (__Block_byref_num1_0 *)&num1, (__Block_byref_num2_1 *)&num2, 570425344));

    ((void (*)(__block_impl *))((__block_impl *)blk)->FuncPtr)((__block_impl *)blk);
    
    return 0;
}
```

我们发现，添加了 `__block` 的局部变量竟然变成了结构体实例，栈上生成该结构体的实例：

```objc
struct __Block_byref_num1_0 {
  void *__isa;
  __Block_byref_num1_0 *__forwarding;
  int __flags;
  int __size;
  int num1;
};
struct __Block_byref_num2_1 {
  void *__isa;
  __Block_byref_num2_1 *__forwarding;
  int __flags;
  int __size;
  int num2;
};
```


## 3. Block 存储域

通过前面说明可知，block 转换为 block 结构体类型的局部变量，即栈上生成的该结构体的实例。通过之前的说明可知 block 也是 Objective-C 对象，该 block 的类为 `_NSConcreteStackBlock`，具体整理如下：

| 类 | 设置对象的存储域 |
| --- | --- |
| _NSConcreteStackBlock | 栈 |
| _NSConcreteGlobalBlock | 程序的数据区域（.data 区） |
| _NSConcreteMallocBlock | 堆 |

##### _NSConcreteStackBlock 类型：

```objc
int main(int argc, const char * argv[]) {
    
    void (^blk)(void) = ^{
        printf("Hello World!");
    };
    blk();
    return 0;
}
```

##### _NSConcreteGlobalBlock 类型：

```objc
void (^blk)(void) = ^{
    printf("Hello World!");
};

int main(int argc, const char * argv[]) {
    
    blk();
    return 0;
}
```

##### _NSConcreteMallocBlock 类型：

实际上，大多数情形下编译器会恰当进行判断，自动生成将 block 从栈上复制到堆上的代码


## 4. GCD 几种常用方法

### 4.1. dispatch\_get\_global\_queue

我们没有必要通过 `dispatch_queue_create` 函数逐个生成队列，只要获取系统标准提供的队列 `Global Dispatch Queue` 即可，获取方法如下：

```objc
// Global Dispatch Queue（高优先级）的获取方法
dispatch_queue_t hightQueue = dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_HIGH, 0);
    
// Global Dispatch Queue（默认优先级）的获取方法
dispatch_queue_t defaultQueue = dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0);
    
// Global Dispatch Queue（低优先级）的获取方法
dispatch_queue_t lowQueue = dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_LOW, 0);
    
// Global Dispatch Queue（后台优先级）的获取方法
dispatch_queue_t backgroundQueue = dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_BACKGROUND, 0);
```    

### 4.2. dispatch\_set\_target\_queue

第一个参数为要设置优先级的 queue，第二个参数是参照物，即将第一个 queue 的优先级和第二个 queue 的优先级设置一样：

```objc
dispatch_queue_t queue = dispatch_queue_create("queue", NULL);
dispatch_queue_t backgroundQueue = dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_BACKGROUND, 0);
dispatch_set_target_queue(queue, backgroundQueue);
```

还有一种使用场景是，使用 `dispatch_set_target_queue` 将多个串行的 `queue` 指定到了同一目标，那么多个串行 `queue` 在目标 `queue` 上就是同步执行了，不再是并行执行：

```objc
// 1.创建目标队列
dispatch_queue_t targetQueue = dispatch_queue_create("target.queue", DISPATCH_QUEUE_SERIAL);
    
// 2.创建 3 个串行队列
dispatch_queue_t queue1 = dispatch_queue_create("queue.1", DISPATCH_QUEUE_SERIAL);
dispatch_queue_t queue2 = dispatch_queue_create("queue.2", DISPATCH_QUEUE_SERIAL);
dispatch_queue_t queue3 = dispatch_queue_create("queue.3", DISPATCH_QUEUE_SERIAL);
    
// 3.将 3 个串行队列分别添加到目标队列
dispatch_set_target_queue(queue1, targetQueue);
dispatch_set_target_queue(queue2, targetQueue);
dispatch_set_target_queue(queue3, targetQueue);
    
dispatch_async(queue1, ^{
    NSLog(@"1 in");
    [NSThread sleepForTimeInterval:3.f];
    NSLog(@"1 out");
});    
dispatch_async(queue2, ^{
    NSLog(@"2 in");
    [NSThread sleepForTimeInterval:2.f];
    NSLog(@"2 out");
});
dispatch_async(queue3, ^{
    NSLog(@"3 in");
    [NSThread sleepForTimeInterval:1.f];
    NSLog(@"3 out");
});
``` 

打印结果为：

```objc
2018-02-02 10:51:04.495511+0800 Test[9744:161033] 1 in
2018-02-02 10:51:07.500984+0800 Test[9744:161033] 1 out
2018-02-02 10:51:07.501277+0800 Test[9744:161033] 2 in
2018-02-02 10:51:09.504112+0800 Test[9744:161033] 2 out
2018-02-02 10:51:09.504318+0800 Test[9744:161033] 3 in
2018-02-02 10:51:10.509577+0800 Test[9744:161033] 3 out
```

### 4.3. dispatch\_apply

```objc
dispatch_queue_t queue = dispatch_get_global_queue(0, 0);
dispatch_apply(10, queue, ^(size_t index) {
    NSLog(@"%zu", index);
});
NSLog(@"done");
```
    
如上代码执行结果为：

```objc
2018-02-02 14:28:35.807653+0800 Test[10129:300472] 0
2018-02-02 14:28:35.807653+0800 Test[10129:300507] 3
2018-02-02 14:28:35.807653+0800 Test[10129:300508] 1
2018-02-02 14:28:35.807828+0800 Test[10129:300472] 4
2018-02-02 14:28:35.807653+0800 Test[10129:300506] 2
2018-02-02 14:28:35.807832+0800 Test[10129:300507] 5
2018-02-02 14:28:35.807845+0800 Test[10129:300508] 6
2018-02-02 14:28:35.807948+0800 Test[10129:300472] 7
2018-02-02 14:28:35.808007+0800 Test[10129:300506] 8
2018-02-02 14:28:35.808077+0800 Test[10129:300507] 9
2018-02-02 14:28:35.808545+0800 Test[10129:300472] done
```

在多线程中执行各个处理的执行时间不定，但是输出结果中最后的 done 必定在最后的位置上，这是因为 `dispatch_apply` 函数会等待全部处理执行结束。例如要对 `NSArray` 类对象的所有元素执行处理时，不必一个一个编写 for 循环部分：

```objc
NSArray *arr = @[@"第一个元素",
                 @"第二个元素",
                 @"第三个元素",
                 @"第四个元素",
                 @"第五个元素"];

dispatch_queue_t queue = dispatch_get_global_queue(0, 0);
dispatch_apply(arr.count, queue, ^(size_t index) {
    NSLog(@"%zu : %@", index, arr[index]);
});
NSLog(@"done");
```

打印结果为：

```objc
2018-02-02 14:35:09.766620+0800 Test[10156:311140] 0 : 第一个元素
2018-02-02 14:35:09.766621+0800 Test[10156:311222] 1 : 第二个元素
2018-02-02 14:35:09.766622+0800 Test[10156:311223] 3 : 第四个元素
2018-02-02 14:35:09.766632+0800 Test[10156:311224] 2 : 第三个元素
2018-02-02 14:35:09.766840+0800 Test[10156:311222] 4 : 第五个元素
2018-02-02 14:35:09.766958+0800 Test[10156:311140] done
```

### 4.4. dispatch\_suspend / dispatch\_resume

`dispatch_suspend` 和 `dispatch_resume` 提供了挂起、恢复队列的功能，简单来说，就是可以暂停、恢复队列上的任务。但是这里的挂起，并不能保证可以立即停止队列上正在运行的 `block`，未执行的 `block` 会被挂起。

```objc
dispatch_queue_t queue = dispatch_queue_create("queue", DISPATCH_QUEUE_SERIAL);

dispatch_async(queue, ^{
    NSLog(@"挂起队列之前，打印后延迟 5 秒");
    sleep(5);
    NSLog(@"挂起队列之前，延迟 5 秒后的打印");
});
dispatch_async(queue, ^{
    NSLog(@"挂起队列之前，打印后延迟 5 秒，再一次");
    sleep(5);
    NSLog(@"挂起队列之前，延迟 5 秒后的打印，再一次");
});
    
NSLog(@"打印之后延迟 1 秒");
sleep(1);
    
NSLog(@"挂起队列");
dispatch_suspend(queue);
    
NSLog(@"挂起队列之后，打印后延迟 10 秒");
sleep(10);
    
NSLog(@"恢复队列");
dispatch_resume(queue);
```

打印结果为：

```objc
2018-02-02 15:00:15.817281+0800 Test[10265:348250] 打印之后延迟 1 秒
2018-02-02 15:00:15.817281+0800 Test[10265:348322] 挂起队列之前，打印后延迟 5 秒
2018-02-02 15:00:16.818663+0800 Test[10265:348250] 挂起队列
2018-02-02 15:00:16.818853+0800 Test[10265:348250] 挂起队列之后，打印后延迟 10 秒
2018-02-02 15:00:20.821085+0800 Test[10265:348322] 挂起队列之前，延迟 5 秒后的打印
2018-02-02 15:00:26.820438+0800 Test[10265:348250] 恢复队列
2018-02-02 15:00:31.826083+0800 Test[10265:348327] 挂起队列之前，延迟 5 秒后的打印，再一次
```

### 4.5. dispatch\_semaphore\_wait / dispatch\_semaphore\_signal

当并行执行的处理更新数据时，会产生数据不一致的情况，有时应该程序还会异常结束。如下例子，不考虑顺序，将所有数据添加到 `NSMutableArray` 中：

```objc
dispatch_queue_t queue = dispatch_get_global_queue(0, 0);
    
NSMutableArray *arr = [NSMutableArray array];
for (int i = 0; i < 100000; i++) {
    dispatch_async(queue, ^{
        [arr addObject:@(i)];
        NSLog(@"已经添加：%d", i);
    });
}
```

打印途中崩溃停止，所以此时我们应该使用 `Dispatch Semaphore` 。`Dispatch Semaphore` 是持有计数的信号，类似于过马路时常用的手旗，可以通过时举起手旗，不可通过时放下手旗。`Dispatch Semaphore` 使用计数来实现该功能，计数为 0 时等待，计数为 1 或者大于 1 时，减去 1 而不等待。

```objc
dispatch_queue_t queue = dispatch_get_global_queue(0, 0);
dispatch_semaphore_t semaphore = dispatch_semaphore_create(1);
    
NSMutableArray *arr = [NSMutableArray array];
for (int i = 0; i < 100000; i++) {
    dispatch_async(queue, ^{
        // 一直等待，直到 semaphore 的计数值达到大于等于 1
        dispatch_semaphore_wait(semaphore, DISPATCH_TIME_FOREVER);
        [arr addObject:@(i)];
        NSLog(@"已经添加：%d", i);
        // 通过该函数将 semaphore 计数值加 1
        dispatch_semaphore_signal(semaphore);
    });
}
```

### 4.6. Dispatch I/O / Dispatch Data

在读取较大的文件时，如果将文件分成合适的大小并使用并行队列读取的话,应该会比一般的读取速度快不少。 在 GCD 当中能实现这一功能的就是 Dispatch I/O 和 Dispatch Data。具体代码实现参照 Apple System Log API 里的[源代码](https://opensource.apple.com/source/Libc/Libc-763.11/gen/asl.c)。


## 5. GCD 的实现

涉及到低层的太多，总结不出来，只能贴上[原文](https://github.com/mayan29/BlogCode/tree/master/【读书笔记】《Objective-C%20高级编程：iOS%20与%20OS%20X%20多线程和内存管理》)了。。。


## 后记

> 去年仅阅读了第一篇章，今年花了 4、5 天时间把整本书全部阅读完成。感觉讲的挺不错的，就是翻译的有点不通顺，废话也略多。感觉比较有用的地方是，block 编译成 c++ 代码后的解析，对 block 低层实现原理豁然开朗。
> 
> 本以为年前没有迭代任务，可以慢慢地看书了，突然要在年前启动一个新项目，有点浮躁了，后面的 GCD 低层看不下去了，心态崩了[衰]
