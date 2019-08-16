---
title: 建造者模式（Builder Pattern）
date: 2019-08-16 20:55:07
tags:
	- Java
	- 设计模式
	- 建造者模式
categories:
	- Java设计模式
---

> 参考书籍：《Java设计模式》 @刘伟

**建造者模式：将一个复杂对象的构建与它的表示分离，使得同样的构建过程可以创建不同的表示。**

## 结构

- **Builder（抽象建造者）**：它为创建一个产品对象的各个部件指定抽象接口，在该接口中一般声明两类方法，一类是`buildPathX()`,它们用于创建复杂对象的各个部件；另一类方法是`getResult()`，它们用于返回复杂对象。Builder既可以是抽象类，也可以是接口。
- **ConcreateBuilder（具体建造者）**：它实现了Builder接口，实现各个部件的具体构造和装配方法，定义并明确所创建的复杂对象，还可以提供一个方法返回创建好的复杂产品对象（该方法也可由抽象建造者实现）。
- **Product（产品）**：它是被构建的复杂对象，包含多个组成部件，具体建造者创建该产品的内部表示并定义它的装配过程。
- **Director（指挥者）**：指挥者又称为导演类，它负责安排复杂对象的建造次序，指挥者与抽象建造者之间存在关联关系，可以在其`construct()`建造方法中调用建造者对象的部件构造和装配方法，完成复杂对象的构建。客户端一般只需要与指挥者进行交互，在客户端确定具体建造者的类型，并实例化具体的建造者对象（也可以通过配置文件和反射机制实现），然后通过指挥者类的构造函数或Setter方法将该对象传入指挥者类中。

## 实现

<!-- more-->

**产品**

```java
package cn.secretseater.designpattern.builderpattern;

public class Product {
    //定义部件，部件可以是任意类型，包括值类型和引用类型
    private String partA;
    private String partB;
    private String partC;

    //省略Getter 和 Setter方法 
    //省略toString方法
}
```

**抽象建造者**

```java
package cn.secretseater.designpattern.builderpattern;

public abstract class Builder {
    //创建产品对象
    protected Product product = new Product();

    abstract void buildPartA();
    abstract void buildPartB();
    abstract void buildPartC();

    //返回产品对象
    public Product getResult(){
        return product;
    }
}
```

**具体建造者**

```java
package cn.secretseater.designpattern.builderpattern;

public class ConcreteBuilder extends Builder {
    @Override
    void buildPartA() {
        product.setPartA("A1");
    }

    @Override
    void buildPartB() {
        product.setPartB("B1");
    }

    @Override
    void buildPartC() {
        product.setPartC("C1");
    }
}
```

**指挥者**

- 它隔离了客户端与创建过程。
- 控制产品对象的创建过程，包括某个`buildPartX()`方法是否被调用以及多个`buildPartX()`方法调用的先后次序等。

```java
public class Director {
    private Builder builder;

    public Director(Builder builder) {
        this.builder = builder;
    }

    public void setBuilder(Builder builder) {
        this.builder = builder;
    }

    //产品的构建与组装方法
    public Product construct(){
        builder.buildPartA();
        builder.buildPartB();
        builder.buildPartC();
        return builder.getResult();
    }
}
```

**示例**

```java
package cn.secretseater.designpattern.builderpattern;

public class BuilderPattern {
    public static void main(String[] args) {
        Builder builder=new ConcreteBuilder();//可通过配置文件实现
        Director director = new Director(builder);
        Product product = director.construct();
        System.out.println(product);
    }
}
```
```
# 运行结果：
Product{partA='A1', partB='B1', partC='C1'}
```

## 指挥者类的深入讨论

### 省略Director

在有些情况下，为了简化系统结构，可以将Director和抽象建造者Builder进行合并，在Builder中提供逐步构建复杂产品对象的`construct()`方法。

```java
package cn.secretseater.designpattern.builderpattern;

public abstract class Builder {
    //创建产品对象
    protected static Product product = new Product();

    abstract void buildPartA();
    abstract void buildPartB();
    abstract void buildPartC();

    public static Product construct(Builder builder){
        builder.buildPartA();
        builder.buildPartB();
        builder.buildPartC();
        return product;
    }
}
```

或者是：

```java
package cn.secretseater.designpattern.builderpattern;

public abstract class Builder {
    //创建产品对象
    protected Product product = new Product();

    abstract void buildPartA();
    abstract void buildPartB();
    abstract void buildPartC();

    public Product construct(){
        this.buildPartA();
        this.buildPartB();
        this.buildPartC();
        return product;
    }
}
```

### 钩子方法的引入

建造者模式除了可以逐步构建一个复杂产品对象外，还可以通过Director类更加精细的控制产品的创建过程，例如增加一类称为钩子方法（Hook Method）的特殊方法来控制是否对某个`buildPartX()`进行调用。

**抽象建造者**

```java
package cn.secretseater.designpattern.builderpattern;

public abstract class Builder {
    //创建产品对象
    protected Product product = new Product();

    abstract void buildPartA();
    abstract void buildPartB();
    abstract void buildPartC();

    //钩子方法
    public boolean isBuildC(){
        return true;
    }

    //返回产品对象
    public Product getResult(){
        return product;
    }
}

```

**具体建造者**

```java
package cn.secretseater.designpattern.builderpattern;

public class ConcreteBuilder extends Builder {
    @Override
    void buildPartA() {
        product.setPartA("A1");
    }

    @Override
    void buildPartB() {
        product.setPartB("B1");
    }

    @Override
    void buildPartC() {
        product.setPartC("C1");
    }

    // 覆盖钩子函数
    @Override
    public boolean isBuildC() {
        return false;
    }
}
```

**指挥者**

```java
package cn.secretseater.designpattern.builderpattern;

public class Director {
    private Builder builder;

    public Director(Builder builder) {
        this.builder = builder;
    }

    public void setBuilder(Builder builder) {
        this.builder = builder;
    }

    //产品的构建与组装方法
    public Product construct(){
        builder.buildPartA();
        builder.buildPartB();
        if(builder.isBuildC()){
            builder.buildPartC();
        }
        return builder.getResult();
    }
}
```

**示例**

```java
package cn.secretseater.designpattern.builderpattern;

public class BuilderPattern {
    public static void main(String[] args) {
        Builder builder=new ConcreteBuilder();//可通过配置文件实现
        Director director = new Director(builder);
        Product product = director.construct();
        System.out.println(product);
    }
}
```
```
# 运行结果：
Product{partA='A1', partB='B1', partC='null'}
```

## 优/缺点及适用环境

### 优点

- 在建造者模式中，客户端不必知道产品内部组成的细节，将产品本身与产品的创建过程解耦，使得相同的创建过程可以创建不同的对象。
- 每一个具体建造者都相互独立，而与其他具体建造者无关，因此可以很便捷的替换具体建造者或增加新的具体建造者，用户使用不同的具体建造者即可得到不同的产品对象。由于指挥者类针对抽象建造者编程，增加新的具体建造者无须修改原有类库的代码，系统扩展方便，符合开闭原则。
- 可以更加精细地控制产品的创建过程。将复杂产品的创建步骤分解在不同的方法中，使得创建过程更加清晰，也更方便使用程序来控制创建过程。

### 缺点

- 建造者模式所创建的产品一般具有较多共同点，其组成部分相似，如果产品之间的差异性很大，例如很多组成部分都不相同，不适合使用建造者模式，因此其使用范围受到一定的限制。
- 如果产品的内部变化复杂，可能会导致需要定义很多具体建造者类来实现这种变化，导致系统变得很庞大，增加系统的理解难度和运行成本。

### 适用环境

- 需要生成的产品对象有复杂的内部结构，这些产品对象通常包含多个成员变量。
- 需要生成的产品对象的属性相互依赖，需要制定其生成顺序。
- 对象的创建过程独立于创建该对象的类。在建造者模式中通过引入指挥者类将创建过程封装在指挥者类中，而不再建造者类和客户类中。
- 隔离复杂对象的创建和使用，并使得相同创建过程可以创建不同的产品。