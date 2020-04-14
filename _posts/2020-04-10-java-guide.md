---
layout: post
title: Java Guide 笔记
date: 2020-04-10
Author: Syaoran
tags: [java, note, javaGuide]
comments: true
---

这是一篇之前浩宁推荐的[Java Guide](https://github.com/Snailclimb/JavaGuide)的读书笔记，我把一些对我有用的内容尽量简略的记录在这里，方便以后复习和查阅。

# 基础

## 基础知识系统总结

### JVM，JDK和JRE

JVM（Java Virtual Machine）是运行Java字节码（.class文件）的虚拟机  
![i](../post_images/jvm.png)

JRE（Java Runtime Environment）是Java运行环境，是运行已编译的Java程序所需的所有内容的集合，包括 Java 虚拟机（JVM），Java 类库，java 命令和其他的一些基础构件。  

JDK（Java Development Kits）是功能齐全的Java SDK，它包含JRE以及编译器（javac）和一些其他工具如javadoc和jdb，它能够创建和编译程序。

> 如果你只是为了运行一下 Java 程序的话，那么你只需要安装 JRE 就可以了。如果你需要进行一些 Java 编程方面的工作，那么你就需要安装 JDK 了。但是，这不是绝对的。有时，即使您不打算在计算机上进行任何 Java 开发，仍然需要安装 JDK。例如，如果要使用 JSP 部署 Web 应用程序，那么从技术上讲，您只是在应用程序服务器中运行 Java 程序。那你为什么需要 JDK 呢？因为应用程序服务器会将 JSP 转换为 Java servlet，并且需要使用 JDK 来编译 servlet。

### Oracle JDK 和 OpenJDK

OpenJDK 是一个参考模型并且是完全开源的，而Oracle JDK 是OpenJDK 的一个实现，并不是完全开源的。

### Java的主类

java的主类是唯一一个包含main()方法的类。

### Java应用程序与小程序的差别

应用程序是从主线程（main()方法）启动。applet小程序没有main()方法，主要是嵌在浏览器页面上运行（调用init()或是run()）来启动。

### override（重写）与overload（重载）

overload是方法名相同，参数类型和顺序不同。构造方法（构造器）可以被overload，不能被override。

### 多态

所谓多态就是指程序中定义的引用变量所指向的具体类型和通过该引用变量发出的方法调用在编程时并不确定，而是在程序运行期间才确定，即一个引用变量到底会指向哪个类的实例对象，该引用变量发出的方法调用到底是哪个类中实现的方法，必须在由程序运行期间才能决定。

在Java中有两种形式可以实现多态：继承（多个子类对同一方法的重写）和接口（实现接口并覆盖接口中同一方法）。

### 在一个静态方法内调用一个非静态成员为什么是非法的? 

由于静态方法可以不通过对象进行调用，因此在静态方法里，不能调用其他非静态变量，也不可以访问非静态变量成员。

### 在Java中定义一个不做事且没有参数的构造方法的作用

Java 程序在执行子类的构造方法之前，如果没有用 super()来调用父类特定的构造方法，则会调用父类中“没有参数的构造方法”。因此，如果父类中只定义了有参数的构造方法，而在子类的构造方法中又没有用 super()来调用父类中特定的构造方法，则编译时将发生错误，因为 Java 程序在父类中找不到没有参数的构造方法可供执行。解决办法是在父类里加上一个不做事且没有参数的构造方法。

### 接口和抽象类的区别

接口是行为的抽象，抽象类是对类的抽象

### 重写equals()方法就一定同时要重写hashCode()方法

因为要求如果两个对象通过调用equals方法是相等的，那么这两个对象调用hashCode方法必须返回相同的整数。

### 序列化与transient关键字

transient 关键字的作用是阻止实例中那些用此关键字修饰的的变量序列化；当对象被反序列化时，被 transient 修饰的变量值不会被持久化和恢复。transient 只能修饰变量，不能修饰类和方法。

### 获取键盘输入的两种方法

- Scanner  
```
Scanner input = new Scanner(System.in);
String s  = input.nextLine();
input.close();
```
- BufferReader  
```
BufferedReader input = new BufferedReader(new InputStreamReader(System.in));
String s = input.readLine();
```

### Collection工具类和Arrays工具类

Collections 工具类常用方法:
- 排序
- 查找,替换操作
- 同步控制(不推荐，需要线程安全的集合类型时请考虑使用 JUC 包下的并发集合)

Arrays工具类常见操作  
- 排序 : sort()
- 查找 : binarySearch()
- 比较: equals()
- 填充 : fill()
- 转列表: asList()
- 转字符串 : toString()
- 复制: copyOf()

























