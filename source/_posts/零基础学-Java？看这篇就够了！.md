---
title: 零基础学 Java？看这篇就够了！
date: 2018-05-04 22:02:13
categories: Java
tags:
---

![image010](/img/img010.png)

## 一、简介

### 1. Java 语言概述

Java 是 SUN（Stanford University Network，斯坦福大学网络公司）1995 年推出的一门高级编程语言，是一种面向 Internet 的编程语言。

### 2. Java 语言的三种技术架构

__Java__ __EE__（Java Platform，Enterprise Edition，企业版），Java 5.0 以前称为 J2EE。Java EE 是在 Java SE 的基础上构建的。简单点说 Java EE 是用来做网站的。

__Java__ __SE__（Java Platform，Standard Edition，标准版），Java 5.0 以前称为 J2SE。它允许开发和部署在桌面、服务器、嵌入式环境和实时环境中使用的 Java 应用程序。简单点说 Java SE 是用来做电脑上运行的软件的。

__Java__ __ME__（Java Platform，Micro Edition，微型版），Java 5.0 以前称为 J2ME。Java ME 为在移动设备和嵌入式设备上运行的应用程序提供环境。简单点说 Java ME 是用来做手机软件的。

### 3. Java 语言特点

__跨平台性__。只需在操作系统上安装 Java 虚拟机（JVM，Java Virtual Machine）即可，由 JVM 来负责 Java 程序在该系统中的运行。

### 4. Java 语言环境搭建

__JRE__（Java Runtime Environment，Java 运行环境）。包括 JVM 和 Java 程序所需的核心类库等。

__JDK__（Java Development Kit，Java 开发工具包）。其中包含 Java 的开发工具（编译工具、打包工具等）和 JRE，所以安装了 JDK 就不用再单独安装 JRE 了。下载地址：[http://www.oracle.com](http://www.oracle.com)

![image011](/img/img011.png)

JRE 仅仅是运行工具，如需要开发工具则需搭建 JDK。JDK 开发完成的 Java 程序，交给 JRE 去运行。


## 二、初体验

### Hello Java

编写如下代码保存为 Demo.java

```java
class Demo {
    public static void main(String[] args) {
        System.out.println("Hello Java");
    }
}
```

使用终端，在 Demo.java 文件当前目录，输入如下命令进行编译：

```java
$ javac Demo.java 
```

编译成功后在 Demo.java 文件当前目录下生成 Demo.class 文件，输入如下命令运行：

```java
$ java Demo
```

输出结果为：

```java
Hello Java
```

### Mac 使用 Sublime Text 3 搭建 Java 环境

Sublime Text 默认的 Java 处理方式仅仅是编译，Build 后会生成一个 .class 文件，而没有去运行，这时候就需要改配置文件达到编译 + 运行的目的。

__1.__ __Tools__ __—>__ __Build__ __System__ __—>__ __New__ __Build__ __System__

![image012](/img/img012.png)

__2.__ __添加如下代码：__

```java
{
"shell_cmd": "javac -encoding utf-8 $file_name && java $file_base_name",
"file_regex": "^ *\\[javac\\] (,+):([0-9]+):() (.*)$",
"selector": "source.java",
"encoding": "utf-8"
}
```

__3.__ __直接保存，名字随意__

![image013](/img/img013.png)

__4.__ __Tools__ __—>__ __Build__ __System__ __—>__ __选择上一步保存的配置文件__

![image014](/img/img014.png)

__5.__ __cmd__ __+__ __B__ __编译运行，就有输出结果了__

![image015](/img/img015.png)


## 三、基本语法

### 1. 名称规范

- 包名：多单词组成时，所有字母都小写 xxxyyyzzz

- 类名、接口名：多单词组成时，所有单词首字母大写 XxxYyyZyy

- 变量名和函数名：多单词组成时，第一个单词首字母小写，第二个单词开始每个单词首字母大写 xxxYyyZzz

- 常量名：多单词组成时，每个单词用下划线连接，所有字母都大写 XXX_YYY_ZZZ

### 2. 基本数据类型

Java 语言是强类型语言，对于每一种数据都定义了明确的具体数据类型，在内存中分配了不同大小的内存空间。

__整数型__：byte、short（2 byte）、int（4 byte）、long（8 byte）

> 1 byte = 8 bit，1 字节 = 8 位
> 
> 1 byte 取值范围是 -128 ~ 127，也就是 -2^7 ~ 2^7-1
> 
> 1 short 取值范围是 -32768 ~ 32767，也就是 -2^15 ~ 2^15-1
> 
> 1 int 取值范围是 -2^31 ~ 2^31-1

__浮点型__：float（4 byte）、double（8 byte）

__字符型__：char（1 byte）

> 1 char 取值范围是 0 ~ 65535，也就是 0 ~ 2^16-1

__布尔型__：boolean（1 byte）

### 3. 逻辑运算符

__&__：无论左边是 true 还是 false，右边都运算；

__&&__：当左边为 false 时，右边不运算；

__|__：无论左边是 true 还是 false，右边都运算；

__||__：当左边为 true 时，右边不运算；

__^__：两边结果相同为 false，两边结果不同为 true

> true ^ ture   = false;
> 
> false ^ false = false;
> 
> true ^ false  = true;
> 
> false ^ true  = true;

> 应用场景
> 
> 1. 一个数异或同一个数两次，结果不变（7^4^4 = 7），一般用于加密解密
> 
> 2. 数值交换
> 
> ```java
> n = n^m;
> m = n^m;
> n = n^m;
> ```
> 
> 这种方法比加减方法好，因为如果 n 和 m 的值非常大，容易超出 int 范围
> 
> ```java
> n = n+m;
> m = n-m;
> n = n-m;
> ```

### 4. 位运算符

__左移__ __/__ __右移__

```java
3 << 2 = 12;  // 相当于 3 乘以 2 ^ 2，3 * 4
6 >> 1 = 3;   // 相当于 6 除以 2 ^ 1，6 / 2
```

> 应用场景：乘法需要转换成二进制位位相乘，比较费事，使用位运算节省时间

__有符号位移__ __/__ __无符号位移__

```java
-6 >> 1;   // 右移后，左边高位用 1 补位
-6 >>> 1;  // 右移后，左边高位用 0 补位
```

__与运算__ __/__ __或运算__ __/__ __异或运算__

```java
6 & 5 = 4;
6 | 5 = 7;
6 ^ 5 = 3;
```

### 5. 循环语句

#### 打印九九乘法表

下面例子不是介绍九九乘法表的算法，而是 `\t` 制表符的应用。如果用空格，打印中在某一行会没有对齐，如果用制表符则不用考虑这个问题：

```java
for (int i = 1; i <= 9; i++) {
    for (int j = 1; j <= i; j++) {
        System.out.println(j + "*" + i + "=" + j*i + "\t");
    }
}
```

#### 跳出多层循环嵌套

如下例子，仅仅打印了 `1*1=1` 就跳出循环了：

```java
w: for (int i = 1; i <= 9; i++) {
    q: for (int j = 1; j <= i; j++) {
           System.out.println(j + "*" + i + "=" + j*i + "\t");
           break w;
       }
   }
```

### 6. 数组

#### 初始化

```java
// 一维数组
int[] arr = new int[3];
int[] arr = {1, 2, 3};

// 二维数组
int[][] arr = new int[3][];  // 二维数组第二维可以不定义长度，默认第一维指向为空
System.out.println(arr[1]);  // 打印结果为 null

int[][] arr = {{1, 2, 3}, {4, 5, 6}, {7, 8, 9}};
```

#### 数组存储原理

```java
int[] x = new int[3];
```

以上面创建数组为例，x 存储到栈内存中，new int[3] 存储到堆内存中。x 存储的不是 new int[3] 本身，而是 new int[3] 的起始内存地址（例如是 0x0079），也就是 x 指向了该内存地址。并且堆内存中，只要被创建，就会有初始化值。

当不想要 x 这个数组了，则将 x 置为 null，x 不指向之前的地址了，数组在堆内存中则不被使用了。java 虚拟机则将该数组视为垃圾，但是不会立刻将其清除掉，会不定时的启动垃圾回收机制，将该数组实体在堆内存中清除。

> 垃圾回收机制不同于 c++，需要程序员自己将堆内存中垃圾清除，java 将这里做了优化。

```java
int[] arr2 = new int[5];
System.out.println(arr2);
```

打印的结果为：`[I@43556938`，`[` 代表数组，`I` 代表 int 数据类型，`@` 右边的为数组内存存放地址，其为哈希算法算出来的哈希值。如果是二维数组，其打印结果为 `[[I@43556938`

#### 数组排序

实际开发中只调用 `Arrays.sort(arr)` 即可实现排序，但是需要声明 `import java.util.Arrays` 包：

```java
import java.util.Arrays;

class Test {
    public static void main(String[] args) {

    int[] arr = {5, 2, 7, 4, 1, 3, 6};
    Arrays.sort(arr);

    System.out.println(Arrays.toString(arr));
    }
}
```

### 7. 进制转换

#### 十进制转换成二进制

```java
// 方法一
class Test {
    public static void main(String[] args) {

        System.out.println(Integer.toBinaryString(6));
    }
}
```

```java
// 方法二
class Test {
    public static void main(String[] args) {

        toBin(6);
    }

    public static void toBin(int num) {

        StringBuffer sb = new StringBuffer();

        while (num > 0) {

            sb.append(num % 2);
            num = num >> 1;
        }

        System.out.println(sb.reverse());
    }
}
```

```java
// 方法三（查表法）
class Test {
    public static void main(String[] args) {

        toBin(6);
    }

    public static void toBin(int num) {

        char[] chars = {'0', '1'};

        StringBuffer sb = new StringBuffer();

        while (num > 0) {

            sb.append(chars[num & 1]);
            num = num >> 1;
        }

        System.out.println(sb.reverse());
    }
}
```

#### 十进制转换成十六进制

```java
// 方法一
class Test {
    public static void main(String[] args) {

        System.out.println(Integer.toHexString(60));
    }
}
```

```java
// 方法二
class Test {
    public static void main(String[] args) {

        toHex(60);
    }

    public static void toHex(int num) {

        StringBuffer sb = new StringBuffer();

        for (int x = 0; x < 8; x++) {

            // 15 的二进制表达形式为 1111，num 和 15 与运算结果为最后四位的值
            int tmp = num & 15;              
            if (tmp > 9) {
                sb.append((char)(tmp - 10 + 'a'));
            } else {
                sb.append(tmp);
            }
            num = num >> 4;
        }

        System.out.println(sb.reverse());
    }
}
```

```java
// 方法三（查表法）
class Test {
    public static void main(String[] args) {

        toHex(60);
        System.out.println(Integer.toHexString(60));
    }

    public static void toHex(int num) {

        char[] chars = {'0', '1', '2', '3', '4', '5', '6', '7', '8', '9', 'a', 'b', 'c', 'd', 'e', 'f'};

        StringBuffer sb = new StringBuffer();

        for (int x = 0; x < 8; x++) {

            sb.append(chars[num & 15]);
            num = num >> 4;
        }

        System.out.println(sb.reverse());
    }
}
```


## 四、函数

### 1. 面向对象

三个特征：__封装__、__继承__、__多态__。

### 2. 变量

成员变量：定义在函数之外，存在堆内存中。
局部变量：定义在函数之内，存在栈内存中。

### 3. 重载

__定义__：在同一个类中，允许存在一个以上的同名函数，只要它们的参数个数或者参数类似不同即可。

__应用场景__：当定义的功能相同，但参与运算的未知内容不同。那么，这时就定义一个函数名称以表示其功能，方便阅读，而通过参数列表的不同来区分多个同名函数。

__判断是否重载__：

```java
void show(int a, char b, double c){}

a.
void show(int x, char y, double z){}  // 没有，因为和原函数一样

b.
void show(int a, double b, char c){}  // 重载，因为参数类型不同   

c.
void show(int a, char b){}  // 重载，因为参数个数不同

d.
int show(int a, char b, double c){}  // 没有，重载和返回值类型没有关系
```

### 4. 私有权限修饰符

__private__：私有权限修饰符，用于修饰类中的成员变量、成员函数。只能在该类中访问，在外部不能访问。

```java
class Person {

    private int age;
    private void run () {
        System.out.println("I`m running.");
    }
}

class Demo {

    public static void main(String[] args) {

        Person p = new Person();
        p.age = 20;
        p.run();
    }
}
```

上面例子中，Person 类的 `age` 不可以赋值，同时 `run()` 方法也不可调用。

### 5. 构造函数

__构造函数__：可以用于给对象进行初始化，当一个类中没有定义构造函数，那么系统会默认给该类加入一个空参数的构造函数。

```java
class Person {

	private String name;
	private int age;

	// 就相当于重写 init 初始化
	Person() {

	}

	Person(String n) {
		this();
		this.name = n;
	}

	Person(String n, int a) {
		this();
		this.name = n;
		this.age = a;
	}

	void say() {
		System.out.println("Person: name = " + name + ", age = " + age);
	}
}

class Demo {

	public static void main(String[] args) {
		
		Person p1 = new Person(); 
		p1.say();

		Person p2 = new Person("mayan");
		p2.say();

		Person p3 = new Person("mayan", 25);
		p3.say();
	}
}
```

打印结果为：

```java
Person: name = null, age = 0
Person: name = mayan, age = 0
Person: name = mayan, age = 25
```
