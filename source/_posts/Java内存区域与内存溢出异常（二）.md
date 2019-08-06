---
title: Java内存区域与内存溢出异常（二）
date: 2019-06-19 20:32:44
tags:
	- Java
	- Java虚拟机
	- JVM
	- OOM
categories:
	- 深入理解Java虚拟机
---

## 方法区和运行时常量池溢出

- 方法区：用云存储已被虚拟机加载的类信息(类名、访问修饰符、常量池、字段描述、方法描述等)、常量、静态变量、即时编译器编译后的代码数据等。虽然JVM规范把它描述为堆的一个逻辑部分，但是它却又一个别名叫做Non-Heap（非堆）目的是为了和Java堆区分开。
- 运行时常量池：用于存放编译器生成的各种字面量和符号引用，这部分内容将在类加载后进入方法区的运行时常量池中存放。
- 运行时常量池也是方法区的一部分。

String.intern()是一个Native方法，它的作用是：如果字符串常量池中已经包含一个等于此String对象的字符串，则返回代表池中这个字符串的String对象；否则将此String对象包含的字符串添加到常量池中，并返回此String对象的引用。

可以通过`-XX:PermSize`和`-XX:MaxPermSize`限制方法区的大小,从而间接限制常量池的容量
    
> 在JDK8之前的HotSpot JVM，存放这些”永久的”的区域叫做“永久代(permanent generation)”。永久代是一片连续的堆空间，在JVM启动之前通过在命令行设置参数`-XX:MaxPermSize`来设定永久代最大可分配的内存空间，默认大小是64M（64位JVM由于指针膨胀，默认是85M）。永久代的垃圾收集是和老年代(old generation)捆绑在一起的，因此无论谁满了，都会触发永久代和老年代的垃圾收集。<br>
> 而在JDK8里面移除了永生代，而对于存放类的元数据的内存大小的设置变为Metaspace参数，可以通过参数`-XX:MetaspaceSize` 和`-XX:MaxMetaspaceSize`设定大小，但如果不指定MaxMetaspaceSize的话，Metaspace的大小仅受限于native memory的剩余大小。也就是说永久代的最大空间一定得有个指定值，而如果MaxPermSize指定不当，就会OOM)。<br>
> 参考：https://blog.csdn.net/canot/article/details/52808630

<!-- more-->

```
package learn.memory;

import java.util.ArrayList;
import java.util.List;

/**
 * VM Args: -XX:PermSize=10M -XX:MaxPermSize=10M
 */
public class RuntimeConstantPoolOOM {
	
	public static void main(String[] args) {
		//使用List保持常量池的引用 避免Full GC回收常量池行为
		List<String> list=new ArrayList<String>();
		//10 mb 的PermSize 在 integer 范围内足够产生OOM了
		int i=0;
		while(true) {
			list.add(String.valueOf(i++).intern());
		}
		
	}
}
```

对于方法区测试，基本思路就是运行时产生大量的类去填满方法区，直至溢出。

借助于CGLib使方法区出现内存溢出异常
> CGlib开源项目： http://cglib.sourceforge.net/ <br>
> 利用Maven命令下载依赖包参考： https://www.cnblogs.com/ddcoder/p/10374660.html

```
package learn.memory;

import java.lang.reflect.Method;

import net.sf.cglib.proxy.Enhancer;
import net.sf.cglib.proxy.MethodInterceptor;
import net.sf.cglib.proxy.MethodProxy;

/**
 * VM Args: -XX:PermSize=10M -XX:MaxPermSize=10M
 *
 */
public class JavaMethodAreaOOM {
	
	static class OOMObject{
		
	}
	
	public static void main(String[] args) {
		while(true) {
			Enhancer enhancer =new Enhancer();
			enhancer.setSuperclass(OOMObject.class);
			enhancer.setUseCache(false);
			enhancer.setCallback(new MethodInterceptor() {
				
				@Override
				public Object intercept(Object obj, Method method, Object[] args, MethodProxy proxy) throws Throwable {
				
					return proxy.invokeSuper(obj, args);
				}
			});
			enhancer.create();
		}
	}
}
```
运行结果：
```
Exception in thread "main" java.lang.OutOfMemoryError: Metaspace
	at net.sf.cglib.core.AbstractClassGenerator.create(AbstractClassGenerator.java:237)
	at net.sf.cglib.proxy.Enhancer.createHelper(Enhancer.java:377)
	at net.sf.cglib.proxy.Enhancer.create(Enhancer.java:285)
	at learn.memory.JavaMethodAreaOOM.main(JavaMethodAreaOOM.java:32)
```
这类场景除了上面提到的程序使用CGLib字节码增强和动态语言之外，常见的还有：
- 大量JSP或动态产生JSP文件的应用（JSP第一次运行时需要编译为Java类）
- 基于OSGi的应用（即使是同一个类文件，被不同的类加载器加载也会被视为不同的类）

## 本机直接内存溢出

- 直接内存(Direct Memory)：并不是虚拟机运行时数据区的一部分也不是JVM规范定义的内存区域。
- DirectMemory容量可以通过`-XX:MaxDirecyMemorySize`指定,如果不指定，则默认与Java堆的最大值（`-Xmx`指定）一样。

> 这个例子在eclipse里不能直接编译，要到项目的属性，Java Compiler，Errors/Warnings中Forbidden reference(access rules)中设置为warning。<br>
> 另外，因为sun.misc.Unsafe包不能直接使用，所有代码里用反射的技巧得到了一个Unsafe的实例。<br>

```
package learn.memory;
import java.lang.reflect.Field;

import sun.misc.Unsafe;
/**
 * VM Args : -Xmx20M -XX:MaxDirecyMemorySize=10M
 *
 */
public class DirectMemoryOOM {
	
	private static final int _1MB= 1024 *1024;
	
	public static void main(String[] args) throws IllegalArgumentException, IllegalAccessException {
		Field unsafeField = Unsafe.class.getDeclaredFields()[0];
		unsafeField.setAccessible(true);
		Unsafe unsafe=(Unsafe)unsafeField.get(null);
		while(true) {
			unsafe.allocateMemory(_1MB);
		}
	}
}
```
运行结果：
```
Exception in thread "main" java.lang.OutOfMemoryError
	at sun.misc.Unsafe.allocateMemory(Native Method)
	at learn.memory.DirectMemoryOOM.main(DirectMemoryOOM.java:18)
```

由DirectMemory导致的内存溢出，一个明显的特征是在Heap Dump文件中不会看见明显的异常，如果发现OOM之后Dump文件很小，而程序又直接或间接使用了NIO，那就可以考虑检查一些是不是这方面的原因。