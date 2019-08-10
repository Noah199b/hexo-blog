---
title: 垃圾收集器（Garbage Collection，GC）与内存分配策略（一）
date: 2019-06-20 20:56:34
tags:
	- Java虚拟机
	- 垃圾收集器（GC）
categories:
	- 深入理解Java虚拟机
---

> - 哪些内存需要回收？
> - 什么时候回收？
> - 如何回收？


# 哪些内存需要回收？

程序计数器、虚拟机栈、本地方法栈这3个区域随线程而生，随线程而灭。每一个栈帧中分配多少内存基本上是在类结构确定下来时就已知的，因此这几个区域的内存分配和回收都是具备确定性的，在这几个区域不需要过多的考虑回收的问题。方法或线程结束，内存回收。

**Java堆和方法区则不一样，只有在程序运行期间才知道会创建哪些对象，这部分内存的分配和回收是动态的，垃圾回收器所关注的是这部分内存。**

# 什么时候回收？

GC回收之前，先判断哪些对象“存活”，哪些对象已经“死去”（即不可能再被任何途径使用的对象）

## 引用计数算法（Reference Counting）

给对象添加一个引用计数器，每当一个地方引用它，计数器值加1；当引用失效时，计数器值减1；任何时刻计数器为0的对象就是不可能再被使用的。

**主流的JVM里没有选用引用计数算法来管理内存，其中最主要的原因是它很难解决对象之间相互循环引用的问题**

<!-- more -->

举例：

```
   objA.instance = objB;
   objB.instance = objA;
```
除此之外，这两个对象再无任何引用，实际上两个对象已经不可能再被访问，但是它们因为互相引用着对方，导致引用的计数器都不为0，于是引用计数算法无法通知GC收集器回收它们。

## 可达性分析算法（Reachability Analysis）

这个算法的基本思路就是通过一系列的称为“GC Roots”的对象作为起始点，从这些节点开始向下搜索，搜索所走过的路径称为引用链（Reference Chain），当一个对象到GC Roots没有任何引用链相连时，则证明此对象是不可用的。

![image](https://upload-images.jianshu.io/upload_images/14340315-2e2cf687b295b7a4.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/563/format/webp)

![image](https://upload-images.jianshu.io/upload_images/14340315-e9a2f8b4c54c13d8.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/494/format/webp)

> 参考： https://www.jianshu.com/p/b79fe594eed3

在Java中可作为GC Roots的对象包括下面几种：

- 虚拟机栈（栈帧中的本地变量表）中引用的对象。
- 方法区中类静态属性引用的对象。
- 方法区中常量引用的对象。
- 本地方法栈中JNI（即一般说的Native方法）引用的对象。

> 参考： https://blog.csdn.net/yulidrff/article/details/85330045 <br>
> https://blog.csdn.net/weixin_34168880/article/details/89580086