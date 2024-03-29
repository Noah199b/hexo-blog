---
title: 垃圾收集器（Garbage Collection，GC）与内存分配策略（三）
date: 2019-06-23 21:05:28
tags:
	- Java虚拟机
	- 垃圾收集器（GC）
categories:
	- 深入理解Java虚拟机
---

## 垃圾回收算法

### 标记-清除算法（Mark-Sweep）

最基础的算法就是“标记-清除”算法如同它的名字一样，算法分为“标记”和“清除”两个阶段：首先标记处所有需要回收的对象，在标记完成后统一回收所有被标记的对象。

> 标记过程之前已经讲过了。

**不足之处：**

- 一个是效率问题，标记和清除两个过程的效率都不高。
- 另一个是空间问题，标记清除之后会产生大量不连续的内存碎片，空间碎太多可能会导致以后程序在运行过程中需要分配较大对象时，无法找到足够的连续内存而不得不提前触发一次垃圾回收动作。

### 复制算法（Copying）

为了解决效率的问题，一种称为“复制”的收集算法出现了。

它可将可用内存按容量划分为大小相等的两块，每次只使用其中的一块。当这一块内存用完了，就将还存活着的对象复制到另一块上面，然后再把已使用过的内存空间一次性清理掉，这样使得每次都是对整个半区进行内存回收，内存分配时也就不用考虑内存碎片等复杂情况，只要一动堆顶指针，按顺序分配内存即可，实现简单，运行高效。

<!-- more-->

**不足之处：**

- 这种算法的代价是将内存缩小为原来的一半。


现在的商业虚拟机都采用这种算法来回收新生代。IBM研究表明，新生代中98%的对象都是“朝生夕死”的，所以并不需要按照1：1的比例来划分内存空间，而是将内存分为较大的Eden空间和两块较小的Survivor空间，每次使用Eden和其中一块Survivor。当回收时，将Eden和Survivor中还存活的对象一次性复制到另外一块Survivor空间上，最后清理掉Eden和刚才使用过的Survivor空间。

HotSpot虚拟机默认的Eden和Survivor的比例是8：1，也就是每次新生代中可用内存空间为整个新生代容量的90%（80% + 10%），只有10%会被“浪费”。

当然，98%对象可回收只是一般情况下的数据，我们每一办法保证每次回收都只有不多于10%的对象存活，当Survivor空间不足时，需要依赖其他内存（这里指老年代）进行分配担保（Handle Promotion）。

> 关于新生代、老年代参考：https://blog.csdn.net/gary0917/article/details/83629597

### 标记-整理算法（Mark-Compact）

复制收集算法在对象存活率较高的时候就要进行较多的复制操作，效率将会变低。更关键的是如果不想浪费50%的空间，就需要有额外的空间进行分配担保，以应对被使用的内存中所有对象都100%存活的极端情况，所以在老年代一般不能直接选用这种算法。

根据老年代的特点，有人提出了一种“标记-整理”算法，标记过程仍与“标记-清除”算法一样，但后续步骤不是直接对可回收的对象进行清理，而是让所有存活对象都向一端移动，然后直接清理掉边界以外的内存。

**不足之处：**

- 标记/整理算法唯一的缺点就是效率也不高，不仅要标记所有存活对象，还要整理所有存活对象的引用地址。从效率上来说，标记/整理算法要低于复制算法。

> 参考：https://www.jianshu.com/p/c8f7fdb57a55

### 分代收集算法（Generational Collection）

当前商业虚拟机的垃圾收集都采用“分代收集”算法。

主要思想就是根据对象存活周期的不同将内存划分为几块，一般是把Java堆分为新生代和老年代，这样就可以根据各个年代的特点采用最适当的收集算法。

- 新生代：每次垃圾收集都会发现大批对象死去，只有少量存活，那就选用复制算法，只需要付出少量存活对象的复制成本就可以完成收集。
- 老年代：因对象存活率高、没有额外的空间对它进行分配担保，就必须使用“标记-清除”或“标记-整理”算法来回收。