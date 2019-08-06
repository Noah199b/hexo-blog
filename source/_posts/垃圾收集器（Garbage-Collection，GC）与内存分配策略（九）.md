---
title: 垃圾收集器（Garbage Collection，GC）与内存分配策略（九）
date: 2019-07-03 21:14:52
tags:
	- Java
	- Java虚拟机
	- JVM
	- 内存分配策略
categories:
	- 深入理解Java虚拟机
---

## 内存分配与回收策略

内存分配的规则不是百分之百固定的，其细节取决于当前使用的是哪一种垃圾收集器组合，还有虚拟机与内存相关参数的设置。

主要验证Serial / Serial Old收集器下（Parnew/ Serial Old收集器的组合的规则也基本一致）的内存分配和回收策略。

### 对象优先在Eden分配

大多数情况下，对象在新生代Eden区分配。当Eden区没有足够的空间分配时，虚拟机将发起一次Minor GC。

虚拟机提供了`-XX:+PrintGCDetails` 这个收集日志参数，告诉虚拟机在发生垃圾收集行为时打印内存回收日志，并且在进程退出时输出当前的内存各区域分配情况。

<!-- more-->

```
package learn.memory;
/**
 * VM Args: -XX:+UseSerialGC -Xms20M -Xmx20M -Xmn10M -XX:+PrintGCDetails -XX:SurvivorRatio=8
 *
 */
public class testAllocation {
	private static final int _1MB = 1024 * 1024;
	public static void main(String[] args) {
		byte[] allocation1,allocation2,allocation3,allocation4;
		allocation1 = new byte[2 *_1MB];
		allocation2 = new byte[2 *_1MB];
		allocation3 = new byte[2 *_1MB]; 
		allocation4 = new byte[4 *_1MB]; //出现一次Minor GC
	}
}
```
运行结果：

```
[GC (Allocation Failure) [DefNew: 7610K->1024K(9216K), 0.0047143 secs] 7610K->3357K(19456K), 0.0047711 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 
[GC (Allocation Failure) [DefNew: 5278K->0K(9216K), 0.0051712 secs] 7612K->7440K(19456K), 0.0051922 secs] [Times: user=0.00 sys=0.00, real=0.01 secs] 
Heap
 def new generation   total 9216K, used 4178K [0x00000000fec00000, 0x00000000ff600000, 0x00000000ff600000)
  eden space 8192K,  51% used [0x00000000fec00000, 0x00000000ff014930, 0x00000000ff400000)
  from space 1024K,   0% used [0x00000000ff400000, 0x00000000ff400000, 0x00000000ff500000)
  to   space 1024K,   0% used [0x00000000ff500000, 0x00000000ff500000, 0x00000000ff600000)
 tenured generation   total 10240K, used 7440K [0x00000000ff600000, 0x0000000100000000, 0x0000000100000000)
   the space 10240K,  72% used [0x00000000ff600000, 0x00000000ffd44080, 0x00000000ffd44200, 0x0000000100000000)
 Metaspace       used 4808K, capacity 4930K, committed 5248K, reserved 1056768K
  class space    used 518K, capacity 561K, committed 640K, reserved 1048576K
```

> 关于Minor GC 和 Full GC
> -  新生代GC（Minor GC）：指发生在新生代的垃圾收集动作，因为Java对象大多都具备朝生夕灭的特性，所有Minor GC非常频繁，一般回收速度也比较快。
> - 老年代GC（Major GC/Full GC）：指发生在老年代的GC，出现Major GC，经常会伴随至少一次的Minor GC（但非绝对的，在Parallel Scavenge收集器的收集策略里就有直接进行Major GC的策略选择过程）。Major GC的速度一般会别Minor GC慢10倍以上。

### 大对象直接进入老年代

所谓的大对象就是值，需要大量连续内存空间的Java对象，最典型的大对象就是那种很长的字符串已经数组。

大对象堆虚拟机的内存分配来说就是一个坏消息，经常出现大对象容易导致内存还有不少空间时就提前触发垃圾收集以获取足够连续空间来“安置”它们。

虚拟机提供了一个`-XX:PretenureSizeThreshold`参数，令大于这个设置值得对象直接在老年代分配。这样做的目的是避免在Eden区与Survivor区之间发生大量的内存复制。

> PretenureSizeThreshold 参数只对Serial和ParNew两款收集器有效，Parallel Scavenge收集器不认识这个参数，Parallel Scavenge 收集器一般并不需要设置。如果遇到必须使用此参数的场合，可以考虑ParNew加CMS的收集器组合。

```
package learn.memory;
/**
 * VM Args: -verbose:gc -Xms20M -Xmx20M -Xmn10M -XX:+PrintGCDetails -XX:SurvivorRatio=8
 *		-XX:PretenureSizeThreshold=3145728
 */
public class testPretenureSizeThreshold {
	private static final int _1MB = 1024 * 1024;
	
	public static void main(String[] args) {
		byte[] allocation1;
		allocation1 = new byte[4 *_1MB];
	}
}
```
运行结果：
```
Heap
 def new generation   total 9216K, used 1148K [0x00000000fec00000, 0x00000000ff600000, 0x00000000ff600000)
  eden space 8192K,  14% used [0x00000000fec00000, 0x00000000fed1f158, 0x00000000ff400000)
  from space 1024K,   0% used [0x00000000ff400000, 0x00000000ff400000, 0x00000000ff500000)
  to   space 1024K,   0% used [0x00000000ff500000, 0x00000000ff500000, 0x00000000ff600000)
 tenured generation   total 10240K, used 4096K [0x00000000ff600000, 0x0000000100000000, 0x0000000100000000)
   the space 10240K,  40% used [0x00000000ff600000, 0x00000000ffa00010, 0x00000000ffa00200, 0x0000000100000000)
 Metaspace       used 2576K, capacity 4486K, committed 4864K, reserved 1056768K
  class space    used 282K, capacity 386K, committed 512K, reserved 1048576K
```