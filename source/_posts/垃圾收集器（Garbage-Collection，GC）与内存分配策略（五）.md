---
title: 垃圾收集器（Garbage Collection，GC）与内存分配策略（五）
date: 2019-06-24 21:08:31
tags:
	- Java
	- Java虚拟机
	- JVM
	- 垃圾收集器（GC）
categories:
	- 深入理解Java虚拟机
---

## 垃圾收集器

这里讨论的收集器基于JDK 1.7 Update 14之后的HotSpot虚拟机。在这个版本中正式提供了商用的G1收集器。

![image](https://images2015.cnblogs.com/blog/467583/201706/467583-20170628093656618-2097152177.png)

> 参考：https://www.cnblogs.com/chengxuyuanzhilu/p/7088316.html

<!-- more-->

### Serial收集器

Serial收集器时最基本、发展历史最悠久的收集器。

这个收集器时一个单线程的收集器，但它的“单线程”的意义并不仅仅说明它只会使用一个CPU或者一条收集线程去完成垃圾收集工作，更重要的是它在进行垃圾收集时，必须暂停其他所有的工作线程，直到它收集结束。

直到现在它依然是Client模式下的默认新生代收集器。它也有着优于其他收集器的地方：简单而高效（与其他收集器的单线程比），对于限定单个CPU的环境来说，Serial收集器由于没有线程交互的开销，专心做垃圾收集自然可以获得最高的单线程收集效率。

### ParNew收集器

ParNew收集器其实就是Serial收集器的多线程版本，除了使用多条线程进行垃圾收集外，其余行为包括Serial收集器可用的所有控制参数（例如：`-XX:SurrvivorRatio`、`-XX:PretenureSizeThreshold`、`-XX:HandlePromotionFailure`等）、收集算法、Stop The World、对象分配规则、回收策略等都与Serial收集器完全一样。

该收集器多运行在Server模式下的虚拟机中首选的新生代收集器，其中一个与性能无关的但很重要的原因是，除了Serial收集器外，目前只有它能与CMS收集器配合工作。

ParNew收集器也是使用`-XX:+UseConcMarkSweepGC`选项后的默认新生代收集器，也可以使用`-XX:+UseParNewGC`选项强制指定它。

ParNew收集器默认开启的收集线程数与CPU的数量相同。可以使用`-XX:ParallelGCThreads`参数来限制垃圾收集的线程数。

> 并行（Parallel）：指多条垃圾收集线程并行工作，但此时用户线程仍处于等待状态。 <br>
> 并发（Concurrent）：指用户线程与垃圾收集线程同时执行（但不一定是并行的，可能会交替执行），用户程序在继续运行，而垃圾收集程序运行与另一个CPU上。

### Parallel Scavenge收集器

Parallel Scavenge收集器是一个新生代收集器，它也是使用复制算法的收集器，又是并行的多线程收集器。

Parallel Scavenge收集器的关注点是达到一个可控制的吞吐量（Throughput）。
所谓吞吐量就是CPU用于运行用户代码的时间与CPU总耗时的比值，即<br>

```math
吞吐量 = 运行用户代码时间 / (运行用户代码时间 + 垃圾收集时间)
```

Parallel Scavenge收集器提供两个参数用于精确控制吞吐量：

- `-XX:MaxGCPauseMillis`：控制最大垃圾收集停顿时间（大于0的毫秒数）
- `-XX:GCTimeRatio`：直接设置吞吐量大小（大于0且小于100的整数，也就是垃圾收集时间占总时间的比率，相当于吞吐量的倒数。默认值99，就是允许最大1%即1/(99+1)的垃圾收集时间）

Parallel Scanvenge收集器也经常被称为“吞吐量优先”收集器。

- `-XX:+UseAdaptiveSizePolicy`：这是一个开关参数，当这个参数打开之后，就不需要手工指定新生代的大小（`-Xmn`）、Eden与Survivor区的比例（`-XX:SurvivorRatio`）、晋升老年代对象大小（`-XX:PretenureSizeThreshold`）等细节参数了,虚拟机会自动根据当前系统的运行情况收集性能监控信息，动态调整这些参数以提供最合适的停顿时间或最大的吞吐量，这种调节方式称为GC自适应的调节策略（GC Ergonomics）。

### Serial Old收集器

Serial Old是Serial是收集器的老年代版本，它同样是一个单线程收集器使用“标记-整理算法”，也是主要在Client模式下的虚拟机使用。

如果在Server模式下，还有两大用途：

- 在JDK 1.5之前的版本中与Parallel Scavenge收集器搭配使用。
- 作为CMS收集器的后备预案，在并收集发生Concurrent Mode Faliure时使用。

### Parallel Old收集器

Parallel Old是Parallel Scavenge收集器的老年代版本，使用多线程和“标记-整理”算法。

- Parallel Scavenge无法与CMS收集器配合使用。
- 由于Serial Old在服务端应用上性能“拖累”，使用Parallel Scavenge也未必能在整体应用上获得吞吐量最大化额效果。

在注重吞吐量以及CPU资源敏感的场合，都可以优先考虑Parallel Scavenge 加Parallel Old收集器。