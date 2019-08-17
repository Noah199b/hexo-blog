---
title: 原型模式（Prototype Pattern）
date: 2019-08-17 12:39:37
tags:
	- Java
	- 设计模式
	- 原型模式
categories:
	- Java设计模式
---

> 参考书籍：《Java设计模式》 @刘伟

**原型模式：使用原型实例指定待创建对象的类型，并且通过复制这个原型来创建新的对象。**

## 实现

- **Portotype（抽象原型类）**：它是声明克隆方法的接口，是所有具体原型类的父类，它可以是抽象类也可以是接口，甚至还可以是具体实现类。
- **ConcretePrototype（具体原型类）**：它实现在抽象原型类中声明的克隆方法，在克隆方法中返回自己的一个克隆对象。
- **Client（客户类）**：在客户类中，让一个原型对象克隆自身从而创建一个新的对象，只需要直接实例化或通过工厂方法等方式创建一个原型对象，再通过调用该对象的克隆方法即可得到多个相同的对象。由于客户类针对抽象原型类Prototype编程，因此用户可以根据需要选择具体原型类，系统具有较好的可扩展性，增加或更换具体原型类都很方便。

## 浅克隆（Shallow Clone）

在浅克隆中，如果原型对象的成员变量是值类型（如int，double，byte，boolean，char等基本数据类型），将复制一份给克隆对象；如果原型对象的成员变量是引用类型（如类，接口，数组等复杂数据类型），则将引用对象的地址复制一份给克隆对象，也就是说原型对象和克隆对象的成员变量指向相同的内存地址。

## 深克隆（Deep Clone）

在深克隆中，无论原型对象的成员变量是值类型还是引用类型，都将复制一份给克隆对象，深克隆将原型对象的所有引用对象也复制一份给克隆对象。

> 关于浅克隆与深克隆参考笔记：[Java中浅克隆与深克隆](https://secretseater.github.io/2019/08/17/Java%E4%B8%AD%E6%B5%85%E5%85%8B%E9%9A%86%E4%B8%8E%E6%B7%B1%E5%85%8B%E9%9A%86/)

## 实现

<!-- more-->

**抽象原型类**

```java
package cn.secretseater.designpattern.prototypepattern;

public abstract class Prototype {
    public abstract  Prototype clone();
}
```

**具体原型类**

```java
package cn.secretseater.designpattern.prototypepattern;

public class ConcretePrototype extends Prototype {
    private String name;
    private int age;

    //省略Getter和Setter
    //省略toString()方法
    
    //克隆方法
    @Override
    public Prototype clone() {
        ConcretePrototype prototype = new ConcretePrototype();//new 一个新对象
        prototype.setAge(this.getAge());
        prototype.setName(this.getName());
        return prototype;
    }
}
```

**示例**

```java
package cn.secretseater.designpattern.prototypepattern;

public class PrototypePattern {
    public static void main(String[] args) {
        ConcretePrototype prototype = new ConcretePrototype();
        prototype.setName("Tom");
        prototype.setAge(20);
        System.out.println("-------------原型对象-------------");
        System.out.println(prototype);

        ConcretePrototype newPrototype = (ConcretePrototype) prototype.clone();
        System.out.println("-------------克隆对象-------------");
        System.out.println(newPrototype);

        newPrototype.setName("Jerry");
        newPrototype.setAge(18);
        System.out.println("修改后......");
        System.out.println("-------------原型对象-------------");
        System.out.println(prototype);
        System.out.println("-------------克隆对象-------------");
        System.out.println(newPrototype);
    }
}
```
```
# 运行结果：
-------------原型对象-------------
ConcretePrototype{name='Tom', age=20}
-------------克隆对象-------------
ConcretePrototype{name='Tom', age=20}
修改后......
-------------原型对象-------------
ConcretePrototype{name='Tom', age=20}
-------------克隆对象-------------
ConcretePrototype{name='Jerry', age=18}
```

## 原型管理器

原型管理器（Prototype）将多个原型对象存储在一个集合中供客户端使用，它是一个专门负责克隆对象的工厂，其中定义了一个集合用于存储原型对象，如果需要某个原型对象的一个克隆，可以通过复制集合中对应的原型对象来获得。

**示例**

```java
package cn.secretseater.designpattern.prototypepattern;

import java.util.HashMap;
import java.util.Map;

public class PrototypeManager {
    private Map<String,Prototype> prototypeMap= new HashMap<String,Prototype>();

    public PrototypeManager(){
        prototypeMap.put("A",new ConcretePrototypeA());
        prototypeMap.put("B",new ConcretePrototypeB());
    }

    public void add(String key, Prototype prototype){
        prototypeMap.put(key,prototype);
    }

    public Prototype get(String key){
        Prototype prototype;
        prototype = prototypeMap.get(key).clone();//通过克隆方法创建新对象
        return prototype;
    }

}
```

## 优/缺点及适用环境

### 优点

- 当创建新的对象实例较为复制时，使用原型模式可以简化对象的创建过程，通过复制一个已有实例可以提高新实例的创建效率。
- 扩展性较好，由于在原型模式中提供了抽象原型类，在客户端可以针对抽象原型类进行编程，而将具体原型类写在配置文件中，增加或减少产品类对原有系统没有任何影响。
- 原型模式提供了简化的创建结构，工厂方法模式常常需要有一个与产品类等级结构相同的工厂等级结构，而原型模式就不需要这样，原型模式中产品的复制时通过封装在原型类中的克隆方法实现的，无须专门的工厂类来创建产品。
- 可以使用深克隆的方式保存对象的状态，使用原型模式将对象复制一份并将其状态保存起来，以便在需要的时候使用（例如恢复到某一历史状态），可辅助实现撤销操作。

### 缺点

- 需要为每一个类配备一个克隆方法，而且该克隆方法位于一个类的内部，当对已有的类进行改造时需要修改源代码，违背了开闭原则。
- 在实现深克隆时需要编写较为复杂的代码，而且当对象之间存在多重的嵌套引用时，为了实现深克隆，每一层对象对应的类都必须支持深克隆，实现起来可能会比较麻烦。

### 适用环境

- 创建新对象成本较大（例如初始化需要占用较长的时间，占用太多的CPU资源或网络资源），新对象可以通过复制已有的对象来获得，如果是相似对象，则可以对其成员变量稍作修改。
- 系统要保存对象的状态，而对象的状态变化很小。
- 需要避免使用分层的工厂类来创建分层次的对象，并且类的实例对象只有一个或很少的几个组合状态，通过复制原型对象得到新实例可能比使用构造函数创建一个新实例更加方便。