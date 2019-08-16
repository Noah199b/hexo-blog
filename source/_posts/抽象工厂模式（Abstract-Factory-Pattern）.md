---
title: 抽象工厂模式（Abstract Factory Pattern）
date: 2019-08-16 17:32:29
tags:
	- Java
	- 设计模式
	- 抽象工厂模式
categories:
	- Java设计模式
---

> 参考书籍：《Java设计模式》 @刘伟

为了更好地理解抽象工厂模式，先引入两个概念。

- **产品等级结构**：产品等级结构即产品的继承结构，例如一个抽象类是电视机，其子类包括海尔电视机、海信电视机、TCL电视机，则抽象电视机与具体品牌的电视机之间构成了一个产品等级结构，抽象类电视机是父类，而具体品牌的电视机是其子类。
- **产品族**：在抽象工厂模式中，产品族是指同一个工厂生产的位于不同产品等级结构中的一组产品，例如海尔电器工厂生产的海尔电视机、海尔冰箱，海尔电视机位于电视机产品等级结构中，海尔冰箱位于冰箱产品等级结构中，海尔电视机、海尔冰箱构成一个产品族。

**抽象工厂模式：提供一个创建一系列相关或依赖对象的接口，而无须指定它们具体的类。**

## 结构

- **AbstractFactory（抽象工厂）**：它声明了一组用于创建一族产品的方法，每一个方法对应一种产品。
- **ConcreteFactory（具体工厂）**：它实现了在抽象工厂中声明的创建产品的方法，生成一组具体产品，这些产品构成了一个产品族，每一个产品都位于某个产品等级结构中。
- **AbstractProduct（抽象产品）**：它为每种产品声明接口，在抽象产品中声明了产品所具有的业务方法。
- **ConcreteProduct（具体产品）**：它定义具体工厂生产的具体产品对象，实现抽象产品接口中声明的方法。

## 实现

<!-- more-->

**抽象工厂**

```java
public interface AbstractFactory{
    public AbstractProductA createProdectA();//工厂方法一
    public AbstractProductB createProdectB();//工厂方法二
    ...
}
```

**具体工厂**

```java
public class ConcreteFactory implements AbstractFactory{
    public AbstractProductA createProdectA(){
        return new ConcreteProductA1();
    };
    public AbstractProductB createProdectB(){
        return new ConcreteProductB1();
    };
    ...
}
```

与工厂方法模式一样，抽象工厂模式也可以为每一种产品提供一组重载的工厂方法，以不同的方式来创建产品。

## 应用实例

**抽象产品**

```java
package cn.secretseater.designpattern.abstractfactorypattern;

/**
 * 按钮接口
 */
public interface Button {
    void display();
}
```

```java
package cn.secretseater.designpattern.abstractfactorypattern;

/**
 * 文本框接口
 */
public interface TextField {
    void display();
}
```

```java
package cn.secretseater.designpattern.abstractfactorypattern;

/**
 * 组合框接口
 */
public interface ComboBox {
    void display();
}
```

**具体产品**

```java
package cn.secretseater.designpattern.abstractfactorypattern;

public class SpringButton implements Button {
    public void display() {
        System.out.println("显示浅绿色按钮。");
    }
}
```

```java
package cn.secretseater.designpattern.abstractfactorypattern;

public class SummerButton implements Button {
    public void display() {
        System.out.println("显示浅蓝色按钮。");
    }
}
```

```java
package cn.secretseater.designpattern.abstractfactorypattern;

public class SpringTextField implements TextField {
    public void display() {
        System.out.println("显示浅绿色文本框。");
    }
}
```

```java
package cn.secretseater.designpattern.abstractfactorypattern;

public class SummerTextField implements TextField {
    public void display() {
        System.out.println("显示浅蓝色文本框。");
    }
}
```

```java
package cn.secretseater.designpattern.abstractfactorypattern;

public class SpringComboBox implements ComboBox {
    public void display() {
        System.out.println("显示浅绿色组合框。");
    }
}
```

```java
package cn.secretseater.designpattern.abstractfactorypattern;

public class SummerComboBox implements ComboBox {
    public void display() {
        System.out.println("显示浅蓝色组合框。");
    }
}
```

**抽象工厂**

```java
package cn.secretseater.designpattern.abstractfactorypattern;

/**
 * 抽象工厂
 */
public interface SkinFactory {
    Button createButton();
    ComboBox createComboBox ();
    TextField createTextField ();
}
```

**具体工厂**

```java
package cn.secretseater.designpattern.abstractfactorypattern;

public class SpringSkinFactory implements SkinFactory {
    public Button createButton() {
        return new SpringButton();
    }

    public ComboBox createComboBox() {
        return new SpringComboBox();
    }

    public TextField createTextField() {
        return new SpringTextField();
    }
}
```

```java
package cn.secretseater.designpattern.abstractfactorypattern;

public class SummerSkinFactory implements SkinFactory {
    public Button createButton() {
        return new SummerButton();
    }

    public ComboBox createComboBox() {
        return new SummerComboBox();
    }

    public TextField createTextField() {
        return new SummerTextField();
    }
}
```

**示例**

```java
package cn.secretseater.designpattern.abstractfactorypattern;

import cn.secretseater.designpattern.util.XMLUtil;

public class AbstractFactoryPattern {
    public static void main(String[] args) {
        SkinFactory factory;
        Button btn;
        TextField tf;
        ComboBox cb;
        factory = (SpringSkinFactory)XMLUtil.getBean(); //反射取得实例
        btn = factory.createButton();
        tf = factory.createTextField();
        cb = factory.createComboBox();
        btn.display();
        tf.display();
        cb.display();
    }
}
```
```
# 运行结果：
显示浅绿色按钮。
显示浅绿色文本框。
显示浅绿色组合框。
```

## 开闭原则的倾斜性

- **增加产品族**：对于增加新的产品族，抽象工厂模式很好地支持了开闭原则，只需要增加具体产品并对应增加一个新的具体工厂，对已有的代码无须做任何修改。
- **增加新的产品等级结构**：对于增加新的产品等级结构，需要修改所有的工厂角色，包括抽象工厂类，在所有的工厂类中都需要增加生成新产品的方法，违背了开闭原则。

## 优/缺点及适用环境

### 优点

- 抽象工厂模式隔离了具体类的生成，使得客户端并不需要知道什么被创建。由于这种隔离，更换一个具体工厂就变得相对容易，所有的具体工厂都实现了抽象工厂中定义的那些公共接口，因此只需要改变具体工厂的实例就可以在某种程度上改变整个软件系统的行为。
- 当一个产品族中的多个对象被设计成一起工作时，它能够保证客户端始终只使用同一个产品族中的对象。
- 增加新的产品族很方便，无须修改已有系统，符合开闭原则。

### 缺点

- 增加新的产品等级结构麻烦，需要对原有系统进行较大的修改，甚至需要修改抽象层代码，这显然会带来较大的不便，违背了开闭原则。

### 适用环境

- 一个系统不应当依赖于产品类实例如何被创建、组合和表达的细节，这对于所有类型的工厂模式都是很重要的，用户无须关心对象的创建过程，将对象的创建和使用解耦。
- 系统中有多于一个的产品族，而每次只使用其中某一产品族。可以通过配置文件等方式来使用户能够动态改变产品族，也可以很方便地增加新的产品族。
- 属于同一个产品族的产品将在一起作用，这一约束必须在系统的设计中体现出来。同一产品族中的产品可以是没有任何关系的对象，但是它们都具有一些共同的约束，如同一操作系统下的按钮和文本框，按钮与文本框之间没有直接关系，但它们都是属于某一操作系统的，此时具有一个共同的约束条件，即操作系统的类型。
- 产品等级结构稳定，在设计完成之后不会向系统中增加新的产品等级结构或者删除已有的产品等级结构。