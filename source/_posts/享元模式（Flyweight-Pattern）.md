---
title: 享元模式（Flyweight Pattern）
date: 2019-10-09 21:15:06
tags:
	- Java
	- 设计模式
	- 享元模式
categories:
	- Java设计模式
---

 > 参考书籍：《Java设计模式》 @刘伟

 享元模式以共享的方式高效地支持大量细粒度对象的重用，享元对象能做到共享的关键是区分了内部状态（Intrinsic State）和外部状态（Extrinsic State）。

 - 内部状态是存储在享元对象内部并且不会随环境改变而改变的状态，内部状态可以共享。
 - 外部状态是随环境变化而改变的、不可以共享的状态。享元对象的外部状态通常由客户端保存，并在享元对象创建之后需要使用的时候再传入到享元对象内部。一个外部状态与另一个外部状态之间是相互独立的。

**享元模式：运用共享技术有效地支持大量细粒度对象的复用。**

## 结构

- **Flyweight（抽象享元类）**：抽象享元类通常是一个接口或抽象类，在抽象享元类中声明了具体享元类公共的方法，这些方法可以向外界提供享元对象的内部数据（内部状态），同时也可以通过这些方法来设置外部数据（外部状态）。
- **ConcreteFlyweight（具体享元类）**：具体享元类实现了抽象享元类，其实例称为享元对象；在具体享元类中为内部状态提供了存储空间。通常可以结合单例模式来设计具体享元类，为每一个具体享元类提供唯一的享元对象。
- **UnsharedConcreteFlyweight（非共享具体享元类）**：并不是所有的抽象享元类的子类都需要被共享，不能被共享的子类可以设计为非共享具体享元类；当需要一个非共享具体享元类的对象时可以直接实例化创建。
- **FlyweightFactory（享元工厂类）**：享元工厂类用于创建并管理享元对象，它针对抽象享元类编程，将各种类型的具体享元对象存储在一个享元池中，享元池一般设计为一个存储“键值对”的集合（也可以是其他类型的集合），可以结合工厂模式进行设计；当用户请求一个具体享元对象时，享元工厂提供一个存储在享元池中已创建的实例或者创建一个新的实例（如果不存在），返回新创建的实例并将其存储在享元池中。

## 实现（应用）

<!-- more-->

**IgoChessman：围棋棋子类，充当抽象享元类**

```java
package cn.secretseater.designpattern.flyweightpattern;

public abstract class IgoChessman {
    public abstract String getColor();

    public void display(){
        System.out.println("棋子颜色："+this.getColor());
    }
}
```

**BlackIgoChessman：黑色棋子类，充当享元具体类**

```java
package cn.secretseater.designpattern.flyweightpattern;

public class BlackIgoChessman extends IgoChessman {
    public String getColor() {
        return "黑色";
    }
}
```

**WhiteIgoChessman：白色棋子类，充当享元具体类**

```java
package cn.secretseater.designpattern.flyweightpattern;

public class WhiteIgoChessman extends IgoChessman {
    public String getColor() {
        return "白色";
    }
}
```

**IgoChessmanFactory：围棋棋子工厂类，充当享元工厂类，使用单例模式对其进行设计**

```java
package cn.secretseater.designpattern.flyweightpattern;

import java.util.Hashtable;

public class IgoChessmanFactory {
    private static IgoChessmanFactory instance = new IgoChessmanFactory();
    private static Hashtable ht;

    public static IgoChessmanFactory getInstance(){
        return instance;
    }

    private  IgoChessmanFactory(){
        ht = new Hashtable();
        IgoChessman black,white;
        black = new BlackIgoChessman();
        ht.put("b",black);
        white = new WhiteIgoChessman();
        ht.put("w",white);
    }

    public static IgoChessman getIgoChessman(String  color){
        return (IgoChessman)ht.get(color);
    }
}
```

**示例**

```java
package cn.secretseater.designpattern.flyweightpattern;

public class FlyweightPattern {
    public static void main(String[] args) {
        IgoChessman b1,b2,b3,w1,w2;
        IgoChessmanFactory factory;

        factory = IgoChessmanFactory.getInstance();

        b1 = factory.getIgoChessman("b");
        b2 = factory.getIgoChessman("b");
        b3 = factory.getIgoChessman("b");
        System.out.println("判断两颗黑子是否相同："+ (b1==b2));

        w1 = factory.getIgoChessman("w");
        w2 = factory.getIgoChessman("w");
        System.out.println("判断两颗白子是否相同："+ (w1==w2));

        b1.display();
        b2.display();
        b3.display();
        w1.display();
        w2.display();
    }
}
```
```
# 运行结果：

判断两颗黑子是否相同：true
判断两颗白子是否相同：true
棋子颜色：黑色
棋子颜色：黑色
棋子颜色：黑色
棋子颜色：白色
棋子颜色：白色
```

## 有外部状态的享元模式

**Coordinates：坐标类**

```java
package cn.secretseater.designpattern.flyweightpattern;

public class Coordinates {
    private int x;
    private int y;

    public Coordinates(int x, int y) {
        this.x = x;
        this.y = y;
    }

    // 省略Getter 和 Setter
}
```

**修改抽象享元类**

```java
package cn.secretseater.designpattern.flyweightpattern;

public abstract class IgoChessman {
    public abstract String getColor();

    public void display(Coordinates coordinates){
        System.out.println("棋子颜色："+this.getColor()+",棋子位置："+coordinates.getX()+","+coordinates.getY());
    }
}
```

**示例**

```java
package cn.secretseater.designpattern.flyweightpattern;

import sun.plugin.dom.core.CoreConstants;

public class FlyweightPattern {
    public static void main(String[] args) {
        IgoChessman b1,b2,b3,w1,w2;
        IgoChessmanFactory factory;

        factory = IgoChessmanFactory.getInstance();

        b1 = factory.getIgoChessman("b");
        b2 = factory.getIgoChessman("b");
        b3 = factory.getIgoChessman("b");
        System.out.println("判断两颗黑子是否相同："+ (b1==b2));

        w1 = factory.getIgoChessman("w");
        w2 = factory.getIgoChessman("w");
        System.out.println("判断两颗白子是否相同："+ (w1==w2));

        b1.display(new Coordinates(1,3));
        b2.display(new Coordinates(2,9));
        b3.display(new Coordinates(2,3));
        w1.display(new Coordinates(3,2));
        w2.display(new Coordinates(2,4));
    }
}
```
```
# 运行结果：

判断两颗黑子是否相同：true
判断两颗白子是否相同：true
棋子颜色：黑色,棋子位置：1,3
棋子颜色：黑色,棋子位置：2,9
棋子颜色：黑色,棋子位置：2,3
棋子颜色：白色,棋子位置：3,2
棋子颜色：白色,棋子位置：2,4
```

## 优/缺点及适用环境

### 优点

- 享元模式可以减少内存中对象的数量，使得相同或者相似对象在内存中只保存一份，从而节约系统资源，提供系统性能。
- 享元模式的外部状态相对独立，而且不会影响其内部状态，从而使享元对象可以在不同的环境中被共享。

### 缺点

- 享元模式使系统变得复杂，需要分离出内部状态和外部状态，从而使得程序逻辑复杂化。
- 为了使对象可以共享，享元模式需要将享元对象的部分状态外部化，而读取外部状态将使运行时间边长。

### 适用环境

- 一个系统有大量相同或者相似的对象，造成内存的大量耗费。
- 对象的大部分状态都可以外部化，可以将这些外部状态传入对象。
- 在使用享元模式时需要维护一个存储享元对象的享元池，而这需要耗费一定系统资源，因此当在需要多次重复使用享元对象时才使用享元模式。