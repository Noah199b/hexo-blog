---
title: 简单工厂模式（Simple Factory Pattern）
date: 2019-08-16 10:49:53
tags:
	- Java
	- 设计模式
	- 简单工厂模式
categories:
	- Java设计模式
---

> 参考书籍：《Java设计模式》 @刘伟 <br>
> 声明：简单工厂模式并不属于GoF的23种经典设计模式，但通常将它作为学习其他工厂模式的基础。


**定义一个工厂类，它可以根据参数的不同返回不同的实例，被创建的实例通常都具有共同的父类。**

由于在简单工厂模式中用于创建实例的方法通常是静态（static）方法，因此简单工厂模式又被称为静态工厂方法（Static Factory Method）模式，它是一种类创建型模式。

## 结构

- **Factory（工厂角色）**：工厂角色即工厂类，它是简单工厂模式的核心，负责实现创建所有产品实例的内部逻辑；工厂类可以被外界直接调用，创建所需的产品对象；在工厂类中提供了静态的工厂方法factoryMethod（），它的返回类型为抽象产品类型Product。
- **Product（抽象产品介角色）**：它是工厂类创建的所有对象的父类，封装了各种产品对象的公有方法，它的引入将提高系统的灵活性，使得在工厂类中只需定义一个通用的工厂方法，因为所有创建的具体产品对象都是其子类对象。
- **ConcreteProduct（具体产品角色）**：它是简单工厂模式的创建目标，所有被创建的对象都充当在这个角色的某个具体类的实例。每一个具体产品角色都继承了抽象产品角色，需要实现在抽象产品中声明的抽象方法。

## 实现

<!-- more-->

**产品抽象类**

```java
package cn.secretseater.designpattern.simplefactorypattern;

public abstract class Product {
    //所有产品类的公共业务方法
    public void methodSame(){
         System.out.println("public method !");
    }
    //声明抽象业务方法
    public abstract void methodDiff();
}
```

**产品实现类**

```java
package cn.secretseater.designpattern.simplefactorypattern;

public class ConcreteProductA extends Product {
    //实现业务方法
    @Override
    public void methodDiff() {
        //业务方法的实现
        System.out.println("i'm ConcreteProductA !");
    }
}
```
```java
package cn.secretseater.designpattern.simplefactorypattern;

public class ConcreteProductB extends Product {
    //实现业务方法
    @Override
    public void methodDiff() {
        //业务方法的实现
        System.out.println("i'm ConcreteProductB !");
    }
}
```

**工厂类**
```java
package cn.secretseater.designpattern.simplefactorypattern;

public class Factory {
    //静态工厂方法
    public static Product getProduct(String arg){
        Product product = null;
        if("A".equalsIgnoreCase(arg)) {
            product = new ConcreteProductA();
        }else if ("B".equalsIgnoreCase(arg)){
            product = new ConcreteProductB();
        }
        return product;
    }
}
```

**示例**
```java
package cn.secretseater.designpattern.simplefactorypattern;

public class SimpleFactoryPattern {
    public static void main(String[] args) {
        Product product1,product2;
        product1 = Factory.getProduct("A");
        product1.methodSame();
        product1.methodDiff();
        product2 = Factory.getProduct("B");
        product2.methodSame();
        product2.methodDiff();
    }
}
```
```
# 运行结果：
public method !
i'm ConcreteProductA !
public method !
i'm ConcreteProductB !
```

所有的工厂模式都强调一点：两个类A和B直接的关系应该仅仅是A创建B或是A使用B，而不能两种关系都有。将对象的创建和使用分离，使得系统更加符合单一职责原则，有利于对功能的复用和系统的维护。

此外，将对象的创建和使用分离还有一个好处：防止用来实例化一个类的数据和代码在多个类中到处都是，可以将有关创建的至少搬移到一个工厂类中。

## 简化

有时候为了简化工厂模式，可以将抽象产品类和工厂类合并，将静态方法移至抽象产品类中。

```java
package cn.secretseater.designpattern.simplefactorypattern;

public abstract class ProductFactory {
    //静态工厂方法
    public static ProductFactory getProduct(String arg){
        ProductFactory product = null;
        if("A".equalsIgnoreCase(arg)) {
            product = new ConcreteProductA();
        }else if ("B".equalsIgnoreCase(arg)){
            product = new ConcreteProductB();
        }
        return product;
    }
    //所有产品类的公共业务方法
    public void methodSame(){
        System.out.println("public method !");
    }
    //声明抽象业务方法
    public abstract void methodDiff();
}
```

## 优/缺点及适用环境

### 优点

- 工厂类包含必要的逻辑判断，可以决定在什么时候创建哪一个产品的实例，客户端可以免除直接创建产品对象的职责，而仅仅“消费”产品，简单工厂模式实现了对象创建和使用的分类。
- 客户端无须知道所创建的具体产品类的类名，只需要知道具体产品类所对应的参数即可，对于一些复杂的类名，通过简单工厂模式可以在一定程度上减少使用者的记忆量。
- 通过引入配置文件，可以在不修改任何客户端代码的情况下更换和增加新的具体产品类，在一定程度上提高了系统的灵活性。

### 缺点

- 由于工厂类集中了所有产品的创建逻辑，职责过重，一旦不能正常工作，整个系统都要受到影响。
- 使用简单工厂模式势必会增加系统中类的个数（引入了新的工厂类），增加系统的复杂度和理解难度。
- 系统扩展困难，一旦添加新产品就不得不修改工厂逻辑，在产品类型较多时有可能造成工厂逻辑过于复杂，不利于系统的扩展。
- 简单工厂模式由于使用了静态工厂方法，造成工厂角色无法形成基于继承的等级结构。

### 适用环境

- 工厂类负责创建的对象比较少，由于创建的对象较少，不会造成工厂方法中的业务逻辑太复杂。
- 客户端只知道传入工厂类的参数，对于如何创建对象并不关心。