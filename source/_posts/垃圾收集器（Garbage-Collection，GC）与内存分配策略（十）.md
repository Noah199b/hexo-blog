---
title: 垃圾收集器（Garbage Collection，GC）与内存分配策略（十）
date: 2019-07-04 21:17:17
tags:
	- Java
	- Java虚拟机
	- JVM
	- 内存分配策略
categories:
	- 深入理解Java虚拟机
---

### 长期存活的对象将进入老年代

虚拟机给每个对象定义了一个对象年龄（Age）计数器。如果对象在Eden出生并经过第一次Minor GC后仍然存活，并且能被Survivor容纳的话，就被移动到Survivor空间中，并且对象年龄设为1,。对象在Survivor区域中每“熬过”一次Minor GC ，年龄就增加1，当它的年龄增加到一定程度的时候（默认为15岁），就将会被晋升到老年代中。对象晋升老年代的年龄阈值，可以通过参数`-XX:MaxTenuringThreshold`设置。

```
package learn.memory;
/**
 * VM Args: -verbose:gc -Xms20M -Xmn10M -Xmx20M -XX:MaxTenuringThreshold=1
 * -XX:PrintGCDetails -XX:SurvivorRatio=8
 *
 */
public class testTenuringThreshold {
	private final static int _1MB = 1024 *1024;
	
	public static void main(String[] args) {
		byte[] allocation1,allocation2,allocation3;
		allocation1=new byte[_1MB / 4 ];
		allocation2=new byte[4 * _1MB];
		allocation3=new byte[4 * _1MB];
		allocation3 = null;
		allocation3=new byte[2 * _1MB];
	}
}
```
<!-- more-->

==-XX:MaxTenuringThreshold=1==运行结果：

```
[GC (Allocation Failure) [DefNew: 5892K->1023K(9216K), 0.0034039 secs] 5892K->1565K(19456K), 0.0039435 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 
[GC (Allocation Failure) [DefNew: 5120K->0K(9216K), 0.0055869 secs] 5661K->5661K(19456K), 0.0056107 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 
Heap
 def new generation   total 9216K, used 6383K [0x00000000fec00000, 0x00000000ff600000, 0x00000000ff600000)
  eden space 8192K,  77% used [0x00000000fec00000, 0x00000000ff23bcb8, 0x00000000ff400000)
  from space 1024K,   0% used [0x00000000ff400000, 0x00000000ff400000, 0x00000000ff500000)
  to   space 1024K,   0% used [0x00000000ff500000, 0x00000000ff500000, 0x00000000ff600000)
 tenured generation   total 10240K, used 5661K [0x00000000ff600000, 0x0000000100000000, 0x0000000100000000)
   the space 10240K,  55% used [0x00000000ff600000, 0x00000000ffb876a0, 0x00000000ffb87800, 0x0000000100000000)
 Metaspace       used 4808K, capacity 4930K, committed 5248K, reserved 1056768K
  class space    used 518K, capacity 561K, committed 640K, reserved 1048576K
```

==-XX:MaxTenuringThreshold=15==运行结果：
```
[GC (Allocation Failure) [DefNew: 5336K->788K(9216K), 0.0081518 secs] 5336K->4884K(19456K), 0.0082528 secs] [Times: user=0.00 sys=0.00, real=0.01 secs] 
Heap
 def new generation   total 9216K, used 7173K [0x00000000fec00000, 0x00000000ff600000, 0x00000000ff600000)
  eden space 8192K,  77% used [0x00000000fec00000, 0x00000000ff23c470, 0x00000000ff400000)
  from space 1024K,  77% used [0x00000000ff500000, 0x00000000ff5c52b0, 0x00000000ff600000)
  to   space 1024K,   0% used [0x00000000ff400000, 0x00000000ff400000, 0x00000000ff500000)
 tenured generation   total 10240K, used 4096K [0x00000000ff600000, 0x0000000100000000, 0x0000000100000000)
   the space 10240K,  40% used [0x00000000ff600000, 0x00000000ffa00010, 0x00000000ffa00200, 0x0000000100000000)
 Metaspace       used 2577K, capacity 4486K, committed 4864K, reserved 1056768K
  class space    used 282K, capacity 386K, committed 512K, reserved 1048576K
```

### 动态对象年龄判断

为了能更好的适应不同程序的内存状况，虚拟机并不是永远地要求对象年龄必须达到了MaxTenuringThreshold才能晋升老年代，而是在Survivor空间中相同年龄所有对象大小的总和大于Survivor空间的一半（假设这些对象的年龄为x，0 < x < 15），年龄大于或等于该年龄（x）的对象就可以直接进入老年代，无须等到MaxTenuringThreshold中的要求的年龄。

```
package learn.memory;
/**
 * VM Args: -verbose:gc -Xms20M -Xmn10M -Xmx20M -XX:MaxTenuringThreshold=15
 * -XX:PrintGCDetails -XX:SurvivorRatio=8
 *
 */
public class testTenuringThreshold2 {
	private final static int _1MB = 1024 *1024;
	
	public static void main(String[] args) {
		byte[] allocation1,allocation2,allocation3,allocation4;
		allocation1=new byte[_1MB / 4 ];
		allocation2=new byte[_1MB / 4 ];
		allocation3=new byte[4 * _1MB];
		allocation4=new byte[4 * _1MB];
		allocation4 = null;
		allocation4=new byte[4 * _1MB];
	}
}
```
运行结果：
```
[GC (Allocation Failure) [DefNew: 5592K->1024K(9216K), 0.0049623 secs] 5592K->5140K(19456K), 0.0050201 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 
[GC (Allocation Failure) [DefNew: 5120K->0K(9216K), 0.0014130 secs] 9236K->5140K(19456K), 0.0014380 secs] [Times: user=0.01 sys=0.00, real=0.00 secs] 
Heap
 def new generation   total 9216K, used 4178K [0x00000000fec00000, 0x00000000ff600000, 0x00000000ff600000)
  eden space 8192K,  51% used [0x00000000fec00000, 0x00000000ff014930, 0x00000000ff400000)
  from space 1024K,   0% used [0x00000000ff400000, 0x00000000ff400000, 0x00000000ff500000)
  to   space 1024K,   0% used [0x00000000ff500000, 0x00000000ff500000, 0x00000000ff600000)
 tenured generation   total 10240K, used 5140K [0x00000000ff600000, 0x0000000100000000, 0x0000000100000000)
   the space 10240K,  50% used [0x00000000ff600000, 0x00000000ffb05258, 0x00000000ffb05400, 0x0000000100000000)
 Metaspace       used 2577K, capacity 4486K, committed 4864K, reserved 1056768K
  class space    used 282K, capacity 386K, committed 512K, reserved 1048576K
```