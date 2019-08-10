---
title: Java内存区域与内存溢出异常（一）
date: 2019-06-18 20:29:49
tags:
	- Java
	- Java虚拟机
	- OOM
categories:
	- 深入理解Java虚拟机
---

# 实战： OutOfMemoryError
> 参考书籍：《深入理解Java虚拟机-JVM高级特性与最佳实践》 第2版

Eclipse Debug/Run 配置
```
-verbose:gc -Xms20M -Xmn10M -Xmx20M 
-XX:+PrintGCDetails -XX:SurvivorRatio=8
```

## Java堆溢出

- JVM所管理的内存中的最大的一块

- 此内存区域的唯一目的就是存放实例对象

- 几乎所有的对象实例都要在堆上分配（JVM规范描述：所有的对象实例以及数组都要在堆上分配，随着技术成熟这个说法已经变得不是那么“绝对”了）

  <!-- more-->

```
package learn.memory;

import java.util.ArrayList;
import java.util.List;
/**
 * VM Args: -Xms20m -Xmx20m
 *          -XX:+HeapDumpOnOutOfMemoryError 内存溢出转储快照便于分析
 *
 */
public class HeapOOM {
	
	static class OOMObject{
		
	}
	
	public static void main(String[] args) {
		List<OOMObject> list=new ArrayList<OOMObject>();
		
		while(true) {
			list.add(new OOMObject());
		}
	}
}
```
运行结果：
```
Exception in thread "main" java.lang.OutOfMemoryError: Java heap space
	at java.util.Arrays.copyOf(Unknown Source)
	at java.util.Arrays.copyOf(Unknown Source)
	at java.util.ArrayList.grow(Unknown Source)
	at java.util.ArrayList.ensureExplicitCapacity(Unknown Source)
	at java.util.ArrayList.ensureCapacityInternal(Unknown Source)
	at java.util.ArrayList.add(Unknown Source)
	at learn.memory.HeapOOM.main(HeapOOM.java:16)
```

可通过Eclipse Memory Analyzer 对dump出的快照进行分析。

> 这里需要理解两个概念，参考：https://blog.csdn.net/qq_33556350/article/details/81411437
> - 内存泄漏（Memory Leak）：是指程序在申请内存后，无法释放已申请的内存空间，导致系统无法及时回收内存并且分配给其他进程使用。通常少次数的内存无法及时回收并不会到程序造成什么影响，但是如果在内存本身就比较少获取多次导致内存无法正常回收时，就会导致内存不够用，最终导致内存溢出。
> - 内存溢出（Memory Overflow）：指程序申请内存时，没有足够的内存供申请者使用，导致数据无法正常存储到内存中。也就是说给你个int类型的存储数据大小的空间，但是却存储一个long类型的数据，这样就会导致内存溢出。

## 虚拟机栈和本地方法溢出

> 参考：https://blog.csdn.net/YChenFeng/article/details/77247807

- 虚拟机栈：是Java执行方法的内存模型，每个方法在执行的同时都会创建一个栈帧用于存储局部变量表、操作数栈、动态链表、方法出口灯信息。每一个方法从调用直至执行完成的过程，就对应着一个栈帧在虚拟机栈中入栈到出栈的过程。
- 本地方法栈：与虚拟机栈作业非常相似，虚拟机栈为虚拟机执行Java方法（也就是字节码）服务，而本地方法栈则为虚拟机使用到的Native方法服务。

对于HotSpot虚拟机（sun公司，也是java默认的虚拟机）来说，虽然`-Xoss`参数（设置本地方法栈大小）存在，但并不生效，栈容量只由`-Xss`参数设定。

关于虚拟机栈和本地方法栈：JVM规范中描述了两种异常：
- 如果线程请求的栈深度大于虚拟机所允许的最大深度，将抛出StackOverflowError异常。
- 如果虚拟机在扩展时无法申请到足够的内存空间，将抛出OutOfMemoryError异常。



```
package learn.memory;
/**
 * VM Args: -Xss128k
 *
 */
public class JavaVMStackSOF {
	 private int stackLength = 1;
	 
	 public void stackLeak() {
		 stackLength++;
		 stackLeak();
	 }
	 
	 public static void main(String[] args) throws  Exception{
		 JavaVMStackSOF oom=new JavaVMStackSOF();
		 try {
			 oom.stackLeak();
		 }catch(Throwable e) {
			 System.err.println("stack length:" + oom.stackLength);
			 throw e;
		 }		
	}
}
```
运行结果：
```
stack length:1002
Exception in thread "main" java.lang.StackOverflowError
	at learn.memory.JavaVMStackSOF.stackLeak(JavaVMStackSOF.java:10)
	at learn.memory.JavaVMStackSOF.stackLeak(JavaVMStackSOF.java:11)
	at learn.memory.JavaVMStackSOF.stackLeak(JavaVMStackSOF.java:11)
	......
```

多线程测试栈溢出导致内存溢出（不建议尝试，会死机）：

```
package learn.memory;
/**
 * VM Args: -Xss2M
 *
 */
public class JavaVMStackOOM {
	
	private void dontStop() {
		while(true) {}
	}
	
	public void stackLeakByThead() {
		while(true) {
			Thread thread=new Thread(new Runnable() {
				
				@Override
				public void run() {
					dontStop();
				}
			});
			thread.start();
		}
		
	}
	
	public static void main(String[] args) {
		JavaVMStackOOM oom=new JavaVMStackOOM();
		oom.stackLeakByThead();
	}
}
```
每个线程都会对应一个栈，栈的生命周期即为线程的生命周期，栈的内存分配取决于JVM的实现，当线程对应的栈分配不到足够的内存时将会抛出 OutOfMemoryError异常。