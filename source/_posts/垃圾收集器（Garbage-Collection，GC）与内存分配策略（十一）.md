---
title: 垃圾收集器（Garbage Collection，GC）与内存分配策略（十一）
date: 2019-07-06 21:21:37
tags:
	- Java虚拟机
	- 内存分配策略
categories:
	- 深入理解Java虚拟机
---

### 空间分配担保

在发生Minor GC之前，虚拟机会检查老年代的最大可用连续空间是否大于新生代所有对象总空间，如果这个条件成立，那么Minor GC可用确保是安全的。

如果不成立，则虚拟机会查看`HandlePromotionFailure`设置值是否允许担保失败。如果允许，那么会继续检查老年代最大可用连续空间是否大于历次晋升到老年代对象的平均大小，如果大于，尝试着进行一次Minor GC，尽管这次Minor GC是有风险的；如果小于，或者`HandlePromotionFailure`设置不允许冒险，那这时也要改为进行一次Full GC。

> **复习：** 新生代使用复制收集算法，但为了内存利用率，只使用其中一个Survivor空间作为轮换备份，因此当出现大量对象在Minor GC后仍然存活的情况（最极端的就是内存回收后新生代这种所有对象都是存活的），就需要老年代进行分配担保，把Survivor无法容纳的对象直接进入老年代。

取平均值进行比较任然是一种动态概率的手段，也就是说，如果某次Minor GC存活后的对象突增，远远高于平均值的话，依然会导致担保失败（Handel Promotion Failure ）。如果出现了HandlePromotionFailure失败，那就只好在失败后重新发起一次Full GC。

JDK 6 Update 24 之后不使用`HandlePromotionFailure`参数了，规则变为只要老年代的连续空间大于新生代对象总大小或者历次晋升的平均值就会进行Minor GC，否则将进行Full GC。

<!-- more-->

```
package learn.memory;
/**
 * VM Args: -verbose:gc -Xms20M -Xmn10M -Xmx20M -XX:MaxTenuringThreshold=1
 * -XX:+PrintGCDetails -XX:SurvivorRatio=8
 *
 */
public class testHandlePromotion {
	private final static int _1MB = 1024 *1024;
	public static void main(String[] args) {
		byte[] allocation1,allocation2,allocation3,
		allocation4,allocation5,allocation6,allocation7;
		allocation1=new byte[2 * _1MB];
		allocation2=new byte[2 * _1MB];
		allocation3=new byte[2 * _1MB];
		allocation1 = null;
		allocation4=new byte[2 * _1MB];
		allocation5=new byte[2 * _1MB];
		allocation6=new byte[2 * _1MB];
		allocation4 = null;
		allocation5 = null;
		allocation6 = null;
		allocation7=new byte[2 * _1MB];
	}
}
```
JDK 1.8 运行结果
```
[GC (Allocation Failure) [DefNew: 7128K->532K(9216K), 0.0083695 secs] 7128K->4628K(19456K), 0.0084854 secs] [Times: user=0.00 sys=0.01, real=0.01 secs] 
[GC (Allocation Failure) [DefNew: 6836K->0K(9216K), 0.0016557 secs] 10932K->4627K(19456K), 0.0016950 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 
Heap
 def new generation   total 9216K, used 2212K [0x00000000fec00000, 0x00000000ff600000, 0x00000000ff600000)
  eden space 8192K,  27% used [0x00000000fec00000, 0x00000000fee290e0, 0x00000000ff400000)
  from space 1024K,   0% used [0x00000000ff400000, 0x00000000ff400000, 0x00000000ff500000)
  to   space 1024K,   0% used [0x00000000ff500000, 0x00000000ff500000, 0x00000000ff600000)
 tenured generation   total 10240K, used 4627K [0x00000000ff600000, 0x0000000100000000, 0x0000000100000000)
   the space 10240K,  45% used [0x00000000ff600000, 0x00000000ffa84f08, 0x00000000ffa85000, 0x0000000100000000)
 Metaspace       used 2577K, capacity 4486K, committed 4864K, reserved 1056768K
  class space    used 282K, capacity 386K, committed 512K, reserved 1048576K
```

> 补充关于Eden 和两块Survivor的运行原理参考： https://blog.csdn.net/lojze_ly/article/details/49456255