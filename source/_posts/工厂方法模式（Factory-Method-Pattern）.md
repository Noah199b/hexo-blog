---
title: 工厂方法模式（Factory Method Pattern）
date: 2019-08-16 12:29:41
tags:	
	- Java
	- 设计模式
	- 工厂方法模式
categories:
	- Java设计模式
---

> 参考书籍：《Java设计模式》 @刘伟 <br>

**定义一个用于创建对象的接口，但是让子类决定将哪一个类实例化。工厂方法模式让一个类的实例化延迟到其子类。**

## 结构

- **Product（抽象产品）**：它是定义产品的接口，是工厂方法模式所创建对象的超类型，也就是产品对象的公共父类。
- **ConcreteProduct（具体产品）**：它实现了抽象产品接口，某种类型的具体产品由专门的具体工厂创建，具体工厂与具体产品之间一一对应。
- **Factory（抽象工厂）**：在抽象工厂类中声明了工厂方法（Factory Method），用于返回一个产品。抽象工厂是工厂方法模式的核心，所有创建对象的工厂类都必须实现该接口。
- **ConcreteFactory（具体工厂）**：它是抽象工厂类的子类，实现了在抽象工厂中声明的工厂方法，并可由客户端调用，返回一个具体产品类的实例。

## 实现

<!-- more-->

****

**产品抽象**

```java
package cn.secretseater.designpattern.factorymethodpattern;

public abstract class Product {
    //所有产品类的公共业务方法
    public void methodSame(){
        System.out.println("public method !");
    }
    //声明抽象业务方法
    public abstract void methodDiff();
}
```

**产品实现**

```java
package cn.secretseater.designpattern.factorymethodpattern;

public class ConcreteFactory implements Factory {
    public Product factoryMethod() {
        return new ConcreteProduct();
    }
}
```

**工厂接口**

```java
package cn.secretseater.designpattern.factorymethodpattern;

public interface Factory {
    Product factoryMethod();
}
```

**工厂实现**

```java
package cn.secretseater.designpattern.factorymethodpattern;

public class ConcreteFactory implements Factory {
    public Product factoryMethod() {
        return new ConcreteProduct();
    }
}
```

**示例**

```java
package cn.secretseater.designpattern.factorymethodpattern;

public class FactoryMethodPattern {
    public static void main(String[] args) {
        Factory factory;
        factory = new ConcreteFactory();//可通过配置文件与反射机制实现
        Product product;
        product = factory.factoryMethod();
        product.methodDiff();
    }
}
```
```
# 运行结果：
i'm ConcreteProduct !
```

可以通过配置文件来存储具体工厂类ConcreteFactory的类名，再通过反射机制创建具体工厂对象，在更换新的具体工厂时无须修改源代码，系统扩展更为方便。

> Java反射机制的笔记：我会单独去写，到时候附上链接。

## 工厂方法的重载

在某些情况下，可以通过多种方式来初始化同一个产品类。

```java
public interface LoggerFactory{
    Logger createLogger();
    Logger createLogger(String arg);
    Logger createLogger(Object obj);
}
```

## 工厂方法的隐藏

有时候，为了进一步简化客户端的使用，还可以对客户端隐藏工厂方法，此时在工厂类中直接调用产品类的业务方法，客户端无须调用工厂方法创建产品对象，直接使用工厂对象即可调用所创建的产品对象中的业务方法。

```java
//将接口改为抽象类
public abstract class LoggerFactory{
    //在工厂方法中直接调用日志记录器类的业务方法writeLog()
    public void writeLog(){
        Logger logger = this.createLogger();
        logger.writeLog();
    }
    
    public abstract Logger createLogger();
}
```

## 优/缺点及适用环境

### 优点

- 在工厂方法模式中，工厂方法用来创建客户所需要的产品，同时还向客户隐藏了那种具体产品类将被实例化这一细节，用户只需关心所需产品对应对的工厂，无须关心创建细节，甚至无须知道具体产品类的类名。
- 基于工厂角色和产品角色的多态性设计师工厂方法模式的关键。它能够让工厂自主确定创建何种产品对象，而如何创建这个对象的细节完全封装在具体工厂内部。工厂方法模式之所以又被称为多态工厂模式，正式因为所有的具体工厂类都具有同一抽象父类。
- 使用工厂方法模式的另一个优点是在系统中加入新产品时无须修改抽象工厂和抽象产品提供的接口，无须修改客户端，也无须修改其他的具体工厂和具体产品，而只要添加一个具体工厂和具体产品即可，这样系统的可扩展性也就变得非常好，完全符合开闭原则。

### 缺点

- 在添加新产品时需要编写新的具体产品类，而且还要提供与之对应的具体工厂类，系统中类的个数将成对增加，在一定程度上增加了系统的复杂度，有更多的类需要编译和运行，会给系统带来一些额外的开销。
- 由于考虑到系统的可扩展性，需要引入抽象层，在客户端代码中均使用抽象层进行定义，增加了系统的抽象性和理解难度。


### 适用环境

- 客户端不知道它锁所需要的对象的类。在工厂方法模式中，客户端不需要知道具体产品类的类名，只需要知道所对应工厂即可，具体产品对象有具体工厂类创建，可将具体工厂类的类名存储在配置文件或数据库中。
- 抽象工厂类通过其子类指定创建那个对象。在工厂方法模式中，对于抽象工厂类只需要提供一个创建产品的接口，而由其子类来确定具体要创建的对象，利用面向对象的多态性和里氏代换原则，在程序运行时子类对象将覆盖父类对象，从而使得系统更容易扩展。