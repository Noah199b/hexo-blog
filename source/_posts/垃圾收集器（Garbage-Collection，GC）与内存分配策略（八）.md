---
title: 垃圾收集器（Garbage Collection，GC）与内存分配策略（八）
date: 2019-07-02 21:12:59
tags:
	- Java虚拟机
	- 垃圾收集器（GC）
categories:
	- 深入理解Java虚拟机
---

### GC日志

每一种收集器的日志形式都是由它们自身的实现所决定的，换而言之，每个收集器的日志格式都可以不一样。但是虚拟机设计者为了方便用户阅读，将各个收集器的日志都维持在一定的共性，例如以下两段典型的GC日志：
```
33.125: [GC [DefNew: 3324K->152k(3715K), 0.0025925 secs] 3324K->152K(11904K), 0.0031680 secs] 

100.667: [Full GC [Tenured: 0K->210K(10240K), 0.0149142 secs] 4603K->210K(19456K), [Perm : 2999K->2999K(21248K)], 0.0150007 secs] [Times: user=0.01 sys=0.00, real=0.02 secs]
```

- 最前面的数字“`33.125:`”和“`100.667:`”表示GC发生的时间，从JVM启动以来经过的秒数。


- GC日志开头的“`[GC`”和“`[Full` GC”说明此次垃圾收集的停顿类型。如果有Full说明这次GC发生了Stop The World的。如果是调用`System.gc()`方法触发的收集，那么在这里将显示“`[Full GC(System)`”。


- 接下来的“`[DefNew`”、“`[Tenured`”、“`[Perm`”表示发生的区域，这里显示的区域名称与使用的GC收集器是密切相关的。例如（老年代同理）：
    - Serial收集器中新生代名称为“Dedault New Generation”，所以显示“`[DefNew`”。
    - ParNew收集器就会变为“`[ParNew`” ，意为“Parallel New Generation”。
    - Parallel Scavenge收集器配套的新生代名称为“PSYoungGen”。


- 方括号内的“`3324K->152K(3712K)`”表示：
```
GC前该内存区域已使用容量 -> GC后该内存区域已使用容量(该内存区域总容量)
```

- 方括号外的“`3324K->152K(11904K)`”表示
```
GC前Java堆已使用容量 -> GC后Java堆已使用容量(Java堆总容量)
```

- “`0.0025925 secs`”表示该内存区域GC所占用的时间，单位是秒。有的收集器会给出更具体的时间数据，如“`[Times: user=0.01 sys=0.00, real=0.02 secs]`”这里的user、sys和real与Linux的time命令所输出的时间一含义一致，分别代表
    - 用户消耗的CPU时间
    - 内核消耗的CPU时间
    - 操作从开始到结束所经过的墙钟时间（Wall Clock Time） 
> CPU时间与墙钟时间的区别是，墙钟时间包括各种非运算的等待耗时，例如等待磁盘I/O，等待线程阻塞，而CPU时间不包括这些耗时，当系统有多个CPU或者多核的话，多线程操作会叠加这些CPU时间，所以user时间超过real时间是完全正常的。

<!-- more-->

### 垃圾收集器参数总结


| 参数                           | 描述                                                         |
| ------------------------------ | ------------------------------------------------------------ |
| UseSerialGC                    | 虚拟机运行在Client模式下的默认值，打开此开关后使用Serial + Serial Old的收集器组合来进行内存回收。 |
| UseParNewGC                    | 打开此开关后，使用ParNew + Serial Old 的收集器组合进行内存回收。 |
| UseConcMarkSweepGC             | 打开此开关后，使用ParNew + CMS + Serial Old的收集器组合进行内存回收。Serial Old将作为CMS收集器出现Concurrent Mode Failure失败后的后备收集器使用。 |
| UseParallelGC                  | 虚拟机在Server模式下的默认值，打开此开关后，使用Parallel Scavenge + Serial Old（PS MarkSweep）的收集器组合进行内存回收。 |
| UseParallelOldGC               | 打开此开关后，使用Parallel Scavenge + Parallel Old的收集器组合进行内存回收。 |
| SurvivorRatio                  | 新生代中Eden 与 Survivor 区域的容量比值，默认为8，代表Eden : Survivor = 8 : 1 |
| PretenureSizeThreshold         | 直接晋升老年代的对象大小，设置这个参数后，大于这个参数的对象将直接在老年代分配。 |
| MaxTenuringThreshold           | 晋升到老年代的对象年龄。每个对象在坚持过Minor GC之后，年龄就增加1，当超过这个参数时就进入老年代。 |
| UseAdaptiveSizePolicy          | 动态调整Java堆中各个区域的大小及进入老年代的年龄。           |
| HandlePromotionFailure         | 是否允许分配担保失败，即老年代的剩余空间不足以应付新生代的整个Eden和Survivor区的所有对象都存活的极端情况。 |
| ParallelGCThreads              | 设置并行GC时进行内存回收的线程数。                           |
| GCTimeRatio                    | GC 时间占总时间的比率，默认是99，即允许1%的GC时间，仅在Parallel Scavenge收集器时生效。 |
| MaxGCPauseMillis               | 设置GC的最大停顿时间。仅在Parallel Scavenge收集器时生效。    |
| CMSInitiatingOccupancyFraction | 设置CMS收集器在老年代空间被使用多少后触发垃圾收集器。默认为68%，仅在CMS收集器中生效。 |
| UseCMSCompactAtFullCollection  | 设置CMS收集器完成垃圾收集后是否要进行一次内存碎片整理。仅在CMS收集器中生效。 |
| CMSFullGCsBeforeCompaction     | 设置CMS收集器在进行若干次垃圾收集后再启动一次内存碎片整理。仅在CMS收集器中生效。 |