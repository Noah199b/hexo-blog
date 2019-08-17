---
title: 单例模式（Singleton Pattern）
date: 2019-08-17 17:46:06
tags:	
	- Java
	- 设计模式
	- 单例模式
categories:
	- Java设计模式
---

> 参考书籍：《Java设计模式》 @刘伟

**单例模式：确保一个类只有一个实例，并提供一个全局访问点来访问这个唯一实例。**

单例模式有三个要点：

- 某个类只能有一个实例
- 它必须自行创建这个实例
- 它必须自行向整个系统提供这个实例

## 结构

对于Singleton（单例），在单例类的内部创建它的唯一实例，并通过静态方法`getInstance()`让客户端可以使用它的唯一实例；为了防止在外部对单例类实例化，将其构造函数的可见性设为private；在单例类内部定义了一个Singleton类型的静态对象作为供外部共享访问的唯一实例。

## 实现

<!-- more-->

```java
package cn.secretseater.designpattern.singletonpattern;

public class Singleton {
    //静态私有成员变量
    private static Singleton instance = null;
    //设置私有构造函数
    private Singleton(){}
    //静态公有工厂方法，返回唯一实例
    public static Singleton getInstance(){
        if(instance == null){
            instance=new Singleton();
        }
        return instance;
    }
}
```
```java
package cn.secretseater.designpattern.singletonpattern;

public class SingletonPattern {
    public static void main(String[] args) {
        Singleton singleton1=Singleton.getInstance();
        Singleton singleton2=Singleton.getInstance();
        //判断两个对象是否相同
        if(singleton1==singleton2){
            System.out.println("两个对象是相同实例");
        }
        else{
            System.out.println("两个对象是不同实例");
        }
    }
}
```
```
# 运行结果：
两个对象是相同实例
```

在单例模式的实现过程中需要注意以下3点：
- 单例类构造函数的可见性为private。
- 提供一个类型为自身的静态私有成员变量。
- 提供一个公有的静态工厂方法。

## 饿汉式单例与懒汉式单例

### 饿汉式单例类

饿汉式单例类（Eager Singeleton）是实现起来最简单的单例类。

```java
package cn.secretseater.designpattern.singletonpattern;

public class EagerSingleton {
    private static final EagerSingleton instance = new EagerSingleton();
    private EagerSingleton(){}
    public static EagerSingleton getInstance(){
        return instance;
    }
}
```

当类被加载时，静态变量instance会被初始化，此时类的私有构造函数会被调用，单例类的唯一实例将被创建。

### 懒汉式单例类

与饿汉式单例相同的是，懒汉式单例类（Lazy Singleton）的构造函数也是私有的。与饿汉式单例类不同的是，懒汉式单例类在第一次被引用时将自己实例化，在懒汉式单例类被加载时不会将自己实例化。

懒汉式单例在第一次调用`getInstance()`方法时实例化，在类加载时并不自行实例化，这种技术又称为延迟加载（Lazy Load）技术，即需要的时候再加载实例。

为了避免多个线程同时调用`getInstance()`方法，可以使用关键字`synchronized`，代码如下：
```java
package cn.secretseater.designpattern.singletonpattern;

public class LazySingleton {
    private static  LazySingleton instance = null;
    private LazySingleton(){}
    //使用synchronized关键字对方法加锁，确保任意时刻只有一个线程可执行该方法
    public  static synchronized  LazySingleton getInstance(){
        if(instance==null){
            instance = new LazySingleton();
        }
        return instance;
    }
}
```

上述代码虽解决了线程安全问题，但是每次调用`getInstance()`方法时都需要进行线程锁定判断，在多线程高并发访问环境中将会导致系统性能大大降低。锁定代码块改进：

```java
package cn.secretseater.designpattern.singletonpattern;

public class LazySingleton {
    private static  LazySingleton instance = null;
    private LazySingleton(){}
    public  static  LazySingleton getInstance(){
        if(instance==null){
            synchronized(LazySingleton.class){
                instance = new LazySingleton();
            }
        }
        return instance;
    }
}
```

### 双重检查锁定

如果使用以上代码来实现单例，还是会存在单例对象不唯一的情况。


线程A、线程B同时通过 `instance==null` 后，线程A抢到执行权创建了实例并返回，释放锁后，线程B不知道instance已经被创建，会再次创建实例并返回，导致产生了多个单例对象，违背了单例模式的设计思想，因此需要进一步改进，在`synchronized`中再进行一次 `instance==null` 判断，这种方式成为双重检查锁定（Double-Check Locking）。

```java
package cn.secretseater.designpattern.singletonpattern;

public class LazySingleton {
    private static volatile  LazySingleton instance = null;
    private LazySingleton(){}
    public  static  LazySingleton getInstance(){
        //第一重判断
        if(instance==null){
            //锁定代码块
            synchronized(LazySingleton.class){
                //第二重判断
                if(instance ==null){
                    instance = new LazySingleton();//创建单例实例
                }
            }
        }
        return instance;
    }
}
```

需要注意的是，如果使用双重检查锁定来实现懒汉式单例类，需要在静态成员变量instance之前增加修饰符`volatile`，被`volatile`修饰的成员变量可以确保多线程都能够正确处理（JDK1.5及以上）。

### 饿汉式单例类与懒汉式单例类的比较

**饿汉式**

- 饿汉式在类加载时实例化，无须考虑多线程问题，可以确保实例唯一性；
- 由于饿汉式一开始就创建对象，因此调用速度和反映时间优于懒汉式；
- 不管系统是否需要该实例，都会在类加载时创建，资源利用率不及懒汉式； 类加载时创建实例，必然导致加载时间边长。

**懒汉式**

- 第一次使用时加载，无须一直占用系统资源；
- 需要考虑多个线程同时访问的问题。

### 使用静态内部类实现单例模式

饿汉式单例类不能实现延迟加载，不管将来用不用始终占据内存；懒汉式单例类线程安全控制繁琐，而且性能受到影响。可见，无论是饿汉式单例还是懒汉式单例都存在一些问题。

为了克服这些问题，在Java语言中可以通过Initialization on Demand Holder（IoDH）技术来实现单例模式。

```java
package cn.secretseater.designpattern.singletonpattern;

public class IoDHSingleton {
    private IoDHSingleton(){}
    private static class HolderClass{
        private  final  static IoDHSingleton instance = new IoDHSingleton();
    }
    public static IoDHSingleton getInstance(){
        return HolderClass.instance;
    }
}
```

由于静态单例对象没有作为Singleton的成员变量直接实例化，因此类加载时不会实例化Singleton，第一次调用`getInstance()`方法时将加载内部类HolderClass，在该内部类中定义了一个static类型的变量instance，此时会首先初始化这个成员变量，由Java虚拟机来保证其线程安全，确保该成员变量只能初始化一次。由于`getInstance()`方法没有任何线程锁定，因此其性能不会造成任何影响。

## 优/缺点及适用环境

### 优点

- 单例模式提供了对唯一实例的受控访问。因为单例类封装了它的唯一实例，所以它可以严格控制客户怎样以及何时访问它。
- 由于在系统内存中只存在一个对象，因此可以节约系统资源，对于一些需要频繁创建和销毁的对象，单例模式无疑可以提高系统的性能。
- 允许可变数目的实例。基于单例模式可以进行扩展，使用与控制单例对象相似的方法来获得指定个数的实例对象，既节省系统资源，又解决了由于单例对象共享过多有损性能问题。（注：自行提供指定书目实例的类可称为多例类。）

### 缺点

- 由于单例模式中没有抽象层，因此单例类的扩展有很大的困难。
- 单例类的职责过重，在一定程度上违背了单一职责原则。因为单例类既提供了业务方法，又提供了创建对象的方法（工厂方法），将对象的创建和对象本身的功能耦合在一起。
- 现在很多面向对象语言（如Java、C#）的运行环境都提供了自动垃圾回收技术，因此如果实例化的共享对象长时间不被利用，系统会认为它是垃圾，会自动销毁并回收资源，下次利用时又将重新实例化，这将导致共享的单例对象状态的丢失。

### 适用环境

- 系统只需要一个实例对象，例如系统要求提供一个唯一的序列号生成器或资源管理器，或者因为资源消耗太大只允许创建一个对象。
- 客户调用类的单个实例只允许使用一个公共访问点，除了该公共访问点，不能通过其他途径访问该实例。


## 拓展

> 枚举实现单例模式，参考： [单例模式-菜鸟教程](https://www.runoob.com/design-pattern/singleton-pattern.html)
```java
public enum Singleton {  
    INSTANCE;  
    public void whateverMethod() {  
    }  
}
```