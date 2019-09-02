---
title: 装饰模式（Decorator Pattern）
date: 2019-09-02 21:50:13
tags:
	- Java
	- 设计模式
	- 装饰模式
categories:
	- Java设计模式
---

 > 参考书籍：《Java设计模式》 @刘伟

 **装饰模式：动态的给一个对象增加一些额外的职责。就扩展功能而言，装饰模式提供了一种比使用子类更加灵活的替代方案。**

 ## 结构

 - **Component（抽象构件）**：它是具体构件和抽象装饰类的共同父类，声明了在具体构件中实现的业务方法，它的引入可以使客户端以一致的方式处理未被装饰的对象以及装饰之后的对象，实现客户端的透明操作。
 - **ConcreteComponent（具体构件）**：它是抽象构件类的子类，用于定义具体的构建对象，实现了在抽象构件中声明的方法，装饰类可以给它增加额外的职责（方法）。
 - **Decorator（抽象装饰类）**：它也是抽象构件类的子类，用于给具体构件增加职责，但是具体职责在其子类中实现。它维护一个指向抽象构件对象的引用，通过该类引用可以调用装饰之前构件对象的方法，并通过其子类扩展该方法，以达到装饰目的。
 - **ConcreteDecorator（具体装饰类）**：它是抽象装饰类的子类，负责向构件添加新的职责。每一个具体装饰类都定义了一些新的行为，它可以调用在抽象装饰类中定义的方法，并可以增加新的方法，用于扩充对象的行为。

## 实现

<!-- more-->

**抽象构件**

```java
package cn.secretseater.designpattern.decoratorpattern;

public abstract class Component {
    public abstract void operation();
}
```

**具体构件**

```java
package cn.secretseater.designpattern.decoratorpattern;

public class ConcreteComponent extends Component {
    public void operation() {
        System.out.println("i'm ConcreteComponent !");
    }
}
```

**抽象装饰类**

```java
package cn.secretseater.designpattern.decoratorpattern;

public class Decorator extends Component{
    //维持一个对抽象构件对象的引用
    private Component component;
    //注入一个抽象类型的对象
    public Decorator(Component component) {
        this.component = component;
    }

    public void operation() {
        //调用原有业务方法
        this.component.operation();
    }
}
```

**具体装饰类**

```java
package cn.secretseater.designpattern.decoratorpattern;

public class ConcreteDecorator extends Decorator {
    public ConcreteDecorator(Component component) {
        super(component);
    }

    public void operation(){
        super.operation();      //调用原有业务方法
        this.addedBehavior();   //调用新增业务方法
    }

    public void addedBehavior(){
        System.out.println("i'm ConcreteDecorator !");
    }
}
```
```java
package cn.secretseater.designpattern.decoratorpattern;

public class ConcreteDecorator1 extends Decorator {
    public ConcreteDecorator1(Component component) {
        super(component);
    }

    public void operation(){
        super.operation();
        this.addedBehavior();
    }

    public void addedBehavior(){
        System.out.println("i'm ConcreteDecorator1 !");
    }
}
```
```java
package cn.secretseater.designpattern.decoratorpattern;

public class ConcreteDecorator2 extends Decorator {
    public ConcreteDecorator2(Component component) {
        super(component);
    }

    public void operation(){
        super.operation();
        this.addedBehavior();
    }

    public void addedBehavior(){
        System.out.println("i'm ConcreteDecorator2 !");
    }
}

```

**示例**

```java
package cn.secretseater.designpattern.decoratorpattern;

public class DecoratorPattern {
    public static void main(String[] args) {
        Component component= new ConcreteDecorator2(new ConcreteDecorator1(new ConcreteComponent()));
        System.out.println("----------装饰1、2----------");
        component.operation();
        Component component1= new ConcreteDecorator2(new ConcreteDecorator(new ConcreteComponent()));
        System.out.println("----------装饰0、2----------");
        component1.operation();
    }
}
```
```
# 运行结果：
----------装饰1、2----------
i'm ConcreteComponent !
i'm ConcreteDecorator1 !
i'm ConcreteDecorator2 !
----------装饰0、2----------
i'm ConcreteComponent !
i'm ConcreteDecorator !
i'm ConcreteDecorator2 !
```

## 透明装饰模式

在标准的装饰模式中，新增行为需在原有业务方法中调用，无论是具体构件对象还是装饰过的构建对象，对于客户端而言都是透明的，这种装饰模式被称为透明（Transparent）装饰模式。

在透明装饰模式中要求客户端完全针对抽象编程，装饰模式的透明性要求客户端程序不应该将对象声明为具体构件类型或具体装饰类型，而应该全部声明为抽象构件类型。对于客户端而言，具体构件对象和具体装饰对象没有任何区别：
```java
    Component c,d;
    c = new ConcreteComponent();
    d = new ConcreteDecorator(c);
    d.operation();
```
而不是：
```java
    ConcreteComponent c = new ConcreteComponent();
    //或
    ConcreteDecorator d = new ConcreteDecorator(c);
```

透明装饰模式可以让客户端透明地使用装饰之前的对象和装饰之后的对象，无须关心它们的区别，此外还可以对一个已装饰过的对象进行多次装饰，得到更加复杂、功能更加强大的对象。

## 半透明装饰模式

在某些情况下，有些新增行为可能需要单独被调用，此时客户端不能再一致性处理装饰之前的对象和装饰之后的对象，这种装饰模式被称为半透明（Semi-transparent）装饰模式。

透明模式的设计难度较大，而且有时需要单独调用新增的业务方法。为了能够调用到新增方法，不得不用具体装饰类型来定义装饰之后的对象，而具体构件类型仍然可以使用抽象构件类型来定义，这种装饰模式即为半透明装饰模式。也就是说，对于客户端而言，具体构件类型无须关心，是透明的；但是具体装饰类型必须指定，这是不透明的。

```java
    //使用抽象构件类型定义
    Component component_o = new ConcreteComponent();
    component_o.operation();
    //使用具体装饰类型定义
    ConcreteDecorator concreteDecorator = new ConcreteDecorator(component_o);
    concreteDecorator.operation();
    //单独调用新增业务方法
    concreteDecorator.addedBehavior();
```

半透明模式可以给系统带来更多的灵活性，设计相对简单，使用起来也非常方便；但是最大的缺点在于不能实现对同一个对象的多次装饰，而且客户端需要区别地对待装饰之前的对象和装饰后的对象。

## 优/缺点及适用环境

### 优点

- 对于扩展一个对象的功能，装饰模式比继承更加灵活，不会导致类的个数急剧增加。
- 可以通过一种动态的方式来扩展一个对象的功能，通过配置文件可以在运行时选择不同的具体装饰类，从而实现不同的行为。
- 可以对一个对象进行多次装饰，通过使用不同的具体装饰类以及这些装饰类的排列组合可以创造出很多不同行为的组合，得到功能更加强大的对象。
- 具体构件类与具体装饰类可以独立变化，用户可以根据需要增加新的具体构件类和具体装饰类，原有类库代码无须改变，符合开闭原则。

### 缺点

- 在使用装饰模式进行系统设计时将产生很多小对象，这些对象的区别在于它们之间相互连接的方式有所不同，而不是它们的类或者属性值有所不同，大量小对象的产生势必会占用更多的系统资源，在一定程度上影响程序的性能。
- 装饰者模式提供了一种比继承更加灵活、机动的解决方案，但同时也意味着比继承更加易于出错，排错也更困难，对于多次装饰的对象，在调试时寻找错误可能需要逐级排查，较为烦琐。

### 适用环境

- 在不影响其他对象的情况下以动态、透明的方式给单个对象添加职责。
- 当不能采用继承的方式对系统进行扩展或采用继承不利于系统扩展和维护时可以使用装饰者模式。不能采用继承的情况主要有两类：
    - 第一类是系统中存在大量独立额扩展，为支持每一种扩展或扩展之间组合将产生大量的子类，使得子类数目呈爆炸性增长；
    - 第二类是因为类已定义为不能继承（例如在Java语言中使用final关键字修饰的类）。