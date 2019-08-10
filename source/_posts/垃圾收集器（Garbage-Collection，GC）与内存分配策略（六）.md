---
title: 垃圾收集器（Garbage Collection，GC）与内存分配策略（六）
date: 2019-06-27 21:10:24
tags:
	- Java虚拟机
	- 垃圾收集器（GC）
categories:
	- 深入理解Java虚拟机
---

### CMS 收集器

CMS(Concurrent Mark Sweep)收集器是一款获取最短回收停顿时间的为目标的收集器。CMS收集器基于“标记-清除”算法实现的。运作过程分为4个步骤：

- **初始标记（CMS initial mark）**：仅仅只是标记下GC Roots能直接关联到的对象，速度很快。
- **并发标记（CMS concurrent mark）**：进行GC Roots Trancing的过程。
- **重新标记（CMS remark）**：为了修正并发期间因用户程序继续运作而导致标记产生变动的那一部分对象的标记记录，这个阶段停顿时间一般会比初始标记时间稍长一点，当远比并发标记的时间短。
- **并发清除（CMS concurrent sweep）**

CMS收集器是一款优秀的收集器：并发收集、低停顿。

Sun公司的一些官方文档中也称之为并发低停顿收集器（Concurrent Low Pause Collector）。

==但是它又以下3个明显的 **缺点**：==

<!-- more-->

#### CMS收集器对CPU资源非常敏感。

其实，面向并发设计的程序都对CPU资源比较敏感。在并发阶段，它虽然不会导致用户线程停顿，但是会因为占用了一部分线程（或者说CPU资源）而导致程序变慢，总吞吐量会降低。<br> 
CMS默认启动的回收线程数是：

```math
(CPU数量+3)/4
```
为了应对这种情况，虚拟机提供了一种称为“增量式并发收集器”（Incremental Concurrent Mark Sweep/i-CMS）的CMS收集器变种，在并发标记、清理的时候让GC线程、用户线程交替运行，尽量减少GC线程的独占资源的时间，这样垃圾收集的过程会很长，但对用户程序的影响就会显得少一些。在目前版本（JDK 1.7）中，i-CMS以及被声明为“deprecated”，即不提倡用户使用。

#### CMS 收集器无法处理浮动垃圾（Floating Garbage），可能出现“Concurrent Mode Failure”失败而导致另一次Full GC的产生。

> 浮动垃圾：由于CMS并发清理阶段用户线程还在运行着，伴随着程序运行自然就还会有新的垃圾产生，这一部分垃圾出现在标记过程后，CMS无法在当次收集中处理掉它们，只好留待下一次GC时再清理掉，这一部分垃圾就称为浮动垃圾。

可通过`-XX:CMSInitiatingOccupancyFractiong`的值来提高CMS收集器触发百分比。

如果CMS运行期间预留的内存无法满足程序需要，就会出现一次“Concurrent Mode Failure”失败，这时虚拟机将启动后备方案：临时起意Serail Old收集器来重新进行老年代垃圾的收集，这样停顿时间就很长了。

#### CMS收集器是基于“标记-清除”算法实现的，这就意味着收集结束时会有大量空间碎片产生。

空间碎片过多时，将会给大对象分配带来很大麻烦，往往会出现在老年代还有很大空间剩余，但是无法找到足够大的连续空间来分配当前对象，不得不提前触发一次Full GC。

为了解决这个问题，CMS收集器提供了`-XX:+UseCMSCompactAtFullCollection`开关参数（默认是开启的），用于在CMS收集器顶不住要进行FullGC时开启内存碎片的合并整理过程，内存整理的过程是无法并发执行的，空间碎片问题没有了，但停顿时间就长了。

虚拟机设计者还提供了另一个参数`-XX:CMSFullGCsBeforeCompaction`，这个参数是用于设置执行多少次不压缩的Full GC后，跟着来一次带压缩的（默认值是0，表示每次进入Full GC都进行碎片整理）。

> 关于Full GC参考：https://www.jianshu.com/p/27703ef3de65