---
title: 桥接模式（Bridge Pattern）
date: 2019-08-19 16:43:00
tags:
	- Java
	- 设计模式
	- 桥接模式
categories:
	- Java设计模式
---

> 参考书籍：《Java设计模式》 @刘伟

**桥接模式：将抽象部分与它的实现部分解耦，使得两者都能够独立变化。**

桥接模式是一种对象结构型模式，它又被称为柄体（Handle and Body）模式或接口（Interface）模式。桥接模式用一种巧妙的方式处理多层继承存在的问题，用抽象关联取代了传统的多层继承，将类之间的静态继承关系转换为动态的对象组合关系，使得系统更加灵活，并易于扩展，同时有效地控制了系统中类的个数。

## 结构

- **Abstraction（抽象类）**：它是用于定义一个抽象类的接口，通常是抽象类而不是接口，其中定义了一个Implementor（实现类接口）类型的对象并可以维护该对象，他与Implementor之间具有关联关系，它既可以包含抽象业务方法，也可以包含具体业务方法。
- **RefinedAbstraction（扩充抽象类）**：它扩充由Abstraction定义的接口，通常情况下它不再是抽象类而是具体类，实现了Abstraction中声明的抽象方法，在RefinedAbstraction中可以调用Implementor中定义的业务方法。
- **Implementor（实现类接口）**：它是定义实现类的接口，这个接口不一定要与Abstraction的接口完全一致，事实上两个接口可以完全不同。一般而言，Implementor接口仅提供基本操作，而Abstraction定义的接口可能会做更多更复杂的操作。Implementor接口对这些基本操作进行了声明，而具体实现交给其子类。通过关联关系，在Abstraction中不仅有自己的方法，还可以调用Implementor中定义的方法，使用关联关系来替代继承关系。
- **ConcreteImplementor（具体实现类）**：它具体实现了Implementor接口，在不同的ConcreteImplementor中提供基本操作的不同实现，在程序运行时ConcreteImplementor对象将替换其父类对象，提供给抽象类具体的业务操作方法。

## 实现

<!-- more-->

桥接模式是一个非常实用的设计模式，在桥接模式中体现了很多面向对象设计原则的思想，包括单一职责原则，开闭原则，合成复用原则，里氏代换原则，依赖倒转原则等。

**实现类接口**

```java
package cn.secretseater.designpattern.bridgepattern;

public interface Implementor {
    void operationImpl();
}
```

**具体实现类**

```java
package cn.secretseater.designpattern.bridgepattern;

public class ConcreteImplementor implements Implementor {
    public void operationImpl() {
        System.out.println("I'm Implementor ！");
    }
}
```

**抽象类**

```java
package cn.secretseater.designpattern.bridgepattern;

public abstract class Abstraction {
    protected Implementor impl;
    public void setImpl(Implementor impl){
        this.impl = impl;
    }
    public abstract void operation();
}
```

**扩充抽象类**

```java
package cn.secretseater.designpattern.bridgepattern;

public class RefinedAbstraction extends  Abstraction {
    public void operation() {
        System.out.println("Abstraction start...");
        impl.operationImpl();
        System.out.println("Abstraction end...");
    }
}
```

**示例**

```java
package cn.secretseater.designpattern.bridgepattern;

public class BridgePattern {
    public static void main(String[] args) {
        Implementor impl = new ConcreteImplementor(); //可通过反射获得
        Abstraction abstraction = new RefinedAbstraction(); //可通过反射获得
        abstraction.setImpl(impl);
        abstraction.operation();
    }
}
```
```
# 运行结果：
Abstraction start...
I'm Implementor ！
Abstraction end...
```

## 桥接模式与适配器模式的联用

桥接模式和适配器模式用于设计的不同阶段，桥接模式用于系统的初步设计，对于存在两个独立变化维度的类可以将其分为抽象和实现化两个角色，使它们可以分别进行变化；而在初步设计完成之后，当发现系统与已有类无法协同工作时可以采用适配器模式。但有时候在设计初期也需要考虑适配器模式，特别是那些涉及大量第三方应用接口额度情况。

## 优/缺点及适用环境

### 优点

- 分离抽象接口及实现部分。桥接模式使用“对象间的关联关系”解耦了抽象和实现之间固有的绑定关系，使得抽象和实现可以沿着各自的维度来变化。所谓抽象和实现沿着各自维度的变化，也就是说抽象和实现不在同一个继承层次结构中，而是“子类化”它们，使它们各自具有自己的子类，以便任意组合子类，从而获得多维度组合对象。
- 在很多情况下，桥接模式可以取代多层次方案，多层继承方案违背了单一职责原则，复用性差，并且类的个数非常多，桥接模式是比多层继承方案更好的解决方法，它极大的减少了子类的个数。
- 桥接模式提高了系统的可扩展性，在两个变化维度中任意扩展一个维度都不需要修改原有系统，符合开闭原则。

### 缺点

- 桥接模式的使用会增加系统的理解与设计难度，由于关联关系建立在抽象层，要求开发者一开始就针对抽象层进行设计与编程。
- 桥接模式要求正确的识别出系统中的两个独立变化的维度，因此其使用范围具有一定的局限性，如何正确识别两个独立维度也需要一定的经验积累。

### 适用环境

- 如果一个系统需要在抽象化和具体化之间增加更多的灵活性，避免在两个层次之间建立静态的继承关系，通过桥接模式可以使它们在抽象层建立一个关联关系。
- 抽象部分和实现部分可以用继承的方式独立扩展而互不影响，在程序运行时可以动态地将一个抽象化子类的对象和一个实现化子类的对象进行组合，即系统需要对抽象化角色和实现化角色进行动态解耦。
- 一个类存在两个（或多个）独立变化的维度，且这两个（或多个）维度都需要独立进行扩展。
- 对于那些不希望使用继承或因为多层继承导致系统类的个数急剧增加的系统，桥接模式尤为使用。