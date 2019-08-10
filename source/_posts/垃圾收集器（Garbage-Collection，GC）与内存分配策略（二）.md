---
title: 垃圾收集器（Garbage Collection，GC）与内存分配策略（二）
date: 2019-06-22 20:59:52
tags:
	- Java虚拟机
	- 垃圾收集器（GC）
categories:
	- 深入理解Java虚拟机
---

## 引用

判定对象是否存活与“引用”有关

- 强引用（Strong reference）：指在程序代码之中普遍存在的，类似于“`Object obj=new Object()`”这类的引用，只有强引用存在，GC就永远不会回收掉被引用的对象。
- 软引用（Soft reference）：用来描述一些还有用但是非必需的对象。对于软引用关联着的对象，在系统将要发生溢出之前，将会把这些对象列进回收范围之中进行第二次回收。如果这次回收还没有足够的内存，才会抛出内存溢出异常。在JDK1.2之后，提供了SoftReference类来实现软引用。
- 弱引用（Weak reference）：也是用来描述非必需对象的。但是它的强度比软引用更弱一些，被弱引用关联的对象只能生存到下一次垃圾回收发生之前。当垃圾收集器工作时，无论当前内存是否足够，都会回收掉只被弱引用关联的对象。在JDK1.2之后，提供了WeakReference类来实现弱引用。
- 虚引用（Phantom reference）：也称为幽灵引用或者幻影引用。它是最弱的一种引用关系。一个对象是否有虚引用的存在，完全不会对其生存时间构成影响，也无法通过虚引用来取得一个对象实例。为一个对象设置虚引用关联的唯一目的就是能在这个对象被收集器回收时收到一个系统通知。在JDK1.2之后，提供了PhantomReference类来实现虚引用。

## 生存还是死亡

即使可达性分析算法中不可达的对象，也并非“非死不可”的，这时候它们出于“缓刑”阶段，要真正宣告一个对象死亡，至少要经历两次标记过程：如果对象在进行可达性分析后发现没有与GC Roots相连接的引用链，那它将会被第一次标记且进行一次筛选，筛选的条件是此对象是否有必要执行`finalize()`方法。当对象没有覆盖`finalize()`方法，或者`finalize()`方法已经被虚拟机调用过，虚拟机将这两种情况都视为“没有必要执行”。

如果这个对象被判定有必要执行`finalize()`方法，那么这个对将会放置在一个叫做F-Queue的队列中，并在稍后由一个由虚拟机自动建立的、低优先级的Finalizer线程去执行它。

这里所谓的“执行”是指虚拟机会触发这个方法，但并不承诺会等待它运行结束，这样做的原因是，如果一个对象在`finalize()`方法中执行缓慢，或者发生了死循环（更极端的情况），将很可能会导致F-Queue队列中其他对象永远处于等待，甚至导致整个内存回收系统崩溃。

`finalize()`方法是对象逃脱死亡命运的最后一次机会，稍后GC将对F-Queue中的对象进行第二次小规模的标记，如果对象要在`finalize()`中成功拯救自己————只要重新与引用链上的任何一个对象建立关联即可，譬如把自己（this关键字）赋值给某个类或者对象的成员变量，那么在第二次标记时它将被移除“即将回收”的集合；如果对象这时候还没有逃脱，那基本上它就真的被回收了。

<!-- more-->

自救演示：
```
package learn.memory;
/**
 * 	此代码演示了两点：
 * 	1.对象可以在GC时自救
 * 	2.这种自救的机会只有一次，因为对象的finalize方法只会被系统自动调用一次
 *
 */
public class FinalizeEscapeGC {
	
	public static FinalizeEscapeGC SAVE_HOOK= null;
	
	public void isAlive() {
		System.err.println("yes,i am still alive!");
	}

	@Override
	protected void finalize() throws Throwable {
		super.finalize();
		System.err.println("finalize method executed");
		FinalizeEscapeGC.SAVE_HOOK = this;
	}
	
	public static void main(String[] args) throws InterruptedException {
		SAVE_HOOK = new FinalizeEscapeGC();
		
		//对象第一次拯救自己
		SAVE_HOOK = null;
		System.gc();
		//因为finalize方法优先级很低，所以暂停0.5秒以等待它
		Thread.sleep(500);
		if (SAVE_HOOK != null) {
			SAVE_HOOK.isAlive();
		}else {
			System.err.println("no , i am dead!");
		}
		//下面这段代码与上面完全相同，但是这次自救却失败了
		SAVE_HOOK = null;
		System.gc();
		
		Thread.sleep(500);
		if (SAVE_HOOK != null) {
			SAVE_HOOK.isAlive();
		}else {
			System.err.println("no , i am dead!");
		}
	}
	
}
```
运行结果：
```
finalize method executed
yes,i am still alive!
no , i am dead!
```

**任何一个对象的`finalize()`方法都只会被系统自动调用一次，如果对象面临下一次回收，它的`finalize()`不会被再次执行。**

## 回收方法区

很多人认为方法区（或者HotSpot的虚拟机中的永久代）是没有垃圾收集的，Java虚拟机规范中确实说过可以不要求虚拟机在方法区实现垃圾收集。

永久代的垃圾收集主要回收两部分内容：废弃常量和无用的类。

### 废弃常量

回收废弃常量与Java堆中的对象非常类似。以常量池中字面量回收为例，加入一个字符串“abc”已经进入了常量池，但是当前系统中没有任何一个String对象是叫做“abc”的，换句话说，就是常量没有任何String对象引用常量池中的“abc”常量，也没有其他地方引用这个字面量，如果这时发生了内存回收，而且必要的话，这个“abc”常量就会被系统清理出常量池。常量池中的其他类（接口）、方法、字段的符号引用也是类似。

### 无用的类

判定一个类是否是“无用的类”的条件则相对苛刻许多，类需要同时满足下面3个条件才能算“无用的类”:

- 该类的所有实例都已经被回收，也就是Java堆中不存在该类的任何实例。
- 加载该类的ClassLoader已经被回收。
- 该类对应的java.lang.Class对象没有在任何地方被引用，无法在任何地方通过反射方位该类的方法。

虚拟机可以对满足上述3个条件的无用类进行回收，这里说的仅仅是“可以”，而并不是和对象一样，不使用了就必然回收。

- `-Xnoclassgc`：关闭虚拟机对class的垃圾回收功能。
- `-verbose:class`：输出虚拟机装入的类的信息
- `-XX:TraceClassLoading`：监控类的加载信息（可以在Product版虚拟机上使用）
- `-XX:TraceClassUnloading`：监控类的卸载信息（需要FastDebug版的虚拟机支持）

**在大量使用反射、动态代理、CGLib等ByteCode框架、动态生存JSP以及OSGI这类频繁自定义ClassLoader的场景中都需要虚拟机具备类卸载的功能，以保证永久代不会溢出。**