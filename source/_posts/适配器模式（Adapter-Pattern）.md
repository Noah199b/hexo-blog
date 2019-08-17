---
title: 适配器模式（Adapter Pattern）
date: 2019-08-17 22:02:20
tags:
	- Java
	- 设计模式
	- 适配器模式
categories:
	- Java设计模式
---

> 参考书籍：《Java设计模式》 @刘伟

**适配器模式：将一个类的接口转换成客户希望的另一个接口。适配器模式让那些接口不兼容的类可以一起工作。**

## 结构

- **Target（目标抽象类）**：目标抽象类定义客户所需要的接口，可以是一个抽象类或接口，也可以是具体类。在类适配器中，由于Java语言不支持多重继承吗，它只能是接口。
- **Adapter（适配器类）**：它可以调用另一个接口，作为一个转换器，对Adaptee和Target进行适配。适配器Adapter是适配器模式的核心，在类适配器中，它通过实现Target接口并继承Adaptee类来使二者产生联系，在对象适配器中，它通过继承Target并关联一个Apaptee对象使二者产生联系。
- **Adaptee（适配者类）**：适配者即被适配的角色，它定义了一个已经存在的接口，这个接口需要适配，适配者类一般是具体类，包含了客户希望使用的业务方法，在某些情况下甚至没有适配者类的源代码。

## 实现

<!-- more-->

### 类适配器

**目标抽象类**

```java
package cn.secretseater.designpattern.adapterpattern;

public interface Target {
    void request();
}
```

**适配者类**

```java
package cn.secretseater.designpattern.adapterpattern;

public class Adaptee {
    public void specificRequest(){
        System.out.println("i'm Adaptee !");
    }
}
```

**适配器类**

```java
package cn.secretseater.designpattern.adapterpattern;

public class Adapter extends Adaptee implements Target {
    public void request() {
        super.specificRequest();
    }
}
```

### 对象适配器

```java
package cn.secretseater.designpattern.adapterpattern;

public class ObjectAdapter implements Target{
    //维持一个适配者对象的引用
    private Adaptee adaptee;

    public ObjectAdapter(Adaptee adaptee) {
        this.adaptee = adaptee;
    }

    public void request() {
        adaptee.specificRequest(); //转发调用
    }
}
```

适配器模式可以将一个类的接口和另一个类的接口匹配起来，使用的前提是不能或不想修改原来的适配者接口和抽象目标类接口。例如购买了一些第三方类库或控件，但是没有源代码，此时使用适配器模式可以统一对象访问接口。

适配器模式更多的是强调对代码的组织，而不是功能的实现。在实际开发中对象适配器的使用频率更高。

## 双向适配器

在对象适配器的使用过程中，如果在适配器中同时包含对目标类和适配者类的引用，适配者可以通过它调用目标类中的方法，目标类也可以通过它调用适配者类中的方法，那么该适配器就是一个双向适配器。

```java
package cn.secretseater.designpattern.adapterpattern;

public class TwoWayAdapter implements  Target,Adaptee {

    private Target target;
    private Adaptee adaptee;

    public TwoWayAdapter(Target target) {
        this.target = target;
    }

    public TwoWayAdapter(Adaptee adaptee) {
        this.adaptee = adaptee;
    }

    public void specificRequest() {
        target.request();
    }

    public void request() {
        adaptee.specificRequest();
    }
}
```

## 优/缺点及适用环境

### 优点

**类适配器模式和对象适配器模式**

- 将目标类和适配类解耦，通过引入一个适配器类来重用现有适配者类，无须修改原有结构。
- 增加了类的透明性和复用性，将具体的业务实现过程封装在适配者类中，对于客户端类而言是透明的，而且提高了适配者的复用性，同一个适配者类可以在多个不同的系统中复用。
- 灵活性和扩展性都非常好，通过使用配置文件可以很方便地更换适配器，也可以在不修改原有代码的基础上增加新的适配器类，完全符合开闭原则。

**类适配器模式**

- 由于适配器类是适配者的子类，因此可以在适配器类中置换一些适配者的方法，使得适配器的灵活性更强。

**对象适配器模式**

- 一个对象适配器可以把多个不同的适配者适配到同一个目标。
- 可以适配一个适配者的子类，由于适配器和适配者之间是关联关系， 根据里氏代换原则，适配者的子类也可通过该适配器进行适配。

### 缺点

**类适配器模式**

- 对于Java、C#等不支持多重继承的语言，一次最多只能适配一个适配者类，不能同时适配多个适配者。
- 适配者类不能为最终类，例如在Java中不能为final类。
- 在Java、C#等语言中，类适配器模式的目标抽象类只能为接口，不能为类，其使用有一定的局限性。

**对象适配器模式**

- 与类适配器相比，在该模式下要在适配器中置换适配者类的某些方法比较麻烦。如果一定要置换掉适配者类的一个或多个方法，可以先做一个适配者类的子类，将适配者类的方法置换掉，然后再把适配者类的子类当成真正的适配者进行适配，实现过程较为复杂。

### 适用环境

- 系统需要使用一些现有的类，而这些类的接口（例如方法名）不符合系统的需求，甚至没有这些类的源码。
- 想创建一个可以重复使用的类，用于和一些彼此之间没有太大关联的类（包括一些可能在将来引进的类）一起工作。

# 缺省适配器模式（Default Adapter Pattern）

缺省适配器模式是适配器模式的一种变体，其应用也较为广泛。

**缺省适配器：当不需要实现一个接口提供的所有方法时，可先设计一个抽象类实现该接口，并为接口中的每一个方法提供一个默认实现（空方法），那么该抽象类和子类可以选择性覆盖父类的某些方法来实现需求，它适用于不想使用一个接口中所有方法的情况，又称为单接口适配器模式。**

## 结构

- **ServiceInterface（适配者接口）**：它是一个接口，通常在该接口中声明了大量方法。
- **AbstractServiceClass（缺省适配器类）**：它是缺省适配器模式的核心类，使用空方法的形式实现了在ServiceInterface接口中声明的方法。通常定义为抽象类，因为对它进行实例化没有任何意义。
- **ConcreteServiceClass（具体业务类）**：它是缺省适配器类的子类，在没有引入适配器之前它需要实现适配者接口，因此需要实现适配者接口中定义的所有方法，而对于一些无须使用的方法不得不提供空实现。在有了缺省适配器之后可以直接继承该适配器类，根据需要有选择性地覆盖在适配器类中定义的方法。

## 实现

**适配者接口**

```java
package cn.secretseater.designpattern.adapterpattern;

public interface ServiceInterface {
    void method1();
    void method2();
    void method3();
    void method4();
    void method5();
}
```

**缺省适配器类**

```java
package cn.secretseater.designpattern.adapterpattern;

public abstract class AbstractServiceClass implements  ServiceInterface{
    public void method1() {}
    public void method2() {}
    public void method3() {}
    public void method4() {}
    public void method5() {}
}
```

**具体业务类**

```java
package cn.secretseater.designpattern.adapterpattern;

public class ConcreteServiceClass extends AbstractServiceClass{
    @Override
    public void method1() {
        System.out.println("i'm method1 !");
    }
}
```

> 参考： [Java设计模式--缺省适配器模式](https://www.cnblogs.com/LUA123/p/7824791.html) · 作者 · 小LUA 