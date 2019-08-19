---
title: 组合模式（Composite Pattern）
date: 2019-08-19 21:31:37
tags:
	- Java
	- 设计模式
	- 组合模式
categories:
	- Java设计模式
---

> 参考书籍：《Java设计模式》 @刘伟

**组合模式：组合多个对象形成树形结构以表示具有部分-整体关系的层次结构。组合模式让客户端可以统一对待单个对象和组合对象。**

组合模式又称为“部分-整体”（Part-Whole）模式，属于对象结构型模式，它将对象组织到树形结构中，可以用来描述整体与部分的关系。

## 结构

- **Component（抽象构件）**：它可以是接口或抽象类，为叶子构件和容器构件声明接口，在该角色中可以包含所有子类共有行为的声明和实现。在抽象构件中定义了访问及管理它的子构件方法，如增加子构件、删除子构件、获取子构件等。
- **Leaf（叶子构件）**：它在组合结构中表示叶子节点对象，叶子节点没有子节点，它实现了在抽象构件中定义的行为。对于那些访问及管理子构件的方法，可以通过抛出异常、提示错误等方式进行处理。
- **Composite（容器构件）**：它在组合中表示容器节点对象，容器节点包含子节点，其子节点可以是叶子节点，也可以是容器节点，它提供一个集合用于存储子节点，实现了在抽象构件中定义的行为，包括那些访问及管理子构件的方法，在其业务方法中可以递归调用其子节点的业务方法。

## 实现

<!-- more-->

**抽象构件**

```java
package cn.secretseater.designpattern.compositepattern;

public abstract class Component {
    public abstract void add(Component c);      //增加成员
    public abstract void remove(Component c);   //删除成员
    public abstract Component getChild(int i);  //获取成员
    public abstract void operation();           //业务方法
}
```

**容器构件**

```java
package cn.secretseater.designpattern.compositepattern;

import java.util.ArrayList;

public class Composite extends Component {

    private ArrayList<Component> list = new ArrayList<Component>();

    public void add(Component c) {
        list.add(c);
    }

    public void remove(Component c) {
        list.remove(c);
    }

    public Component getChild(int i) {
        return (Component)list.get(i);
    }

    public void operation() {
        //容器构件具体业务方法的实现，将递归调用成员构件的业务方法
        for(Object obj: list){
            ((Component)obj).operation();
        }
    }
}
```

**叶子构件**

```java
package cn.secretseater.designpattern.compositepattern;

import java.util.Random;

/**
 * 叶子构件不能添加成员、删除成员、删除成员
 */
public class Leaf extends Component {
    private  int flag = new Random().nextInt(10);
    public void add(Component c) {
        //异常处理或错误提示
        throw  new RuntimeException("叶子构件不能添加成员");
    }

    public void remove(Component c) {
        //异常处理或错误提示
        throw  new RuntimeException("叶子构件不能删除成员");
    }

    public Component getChild(int i) {
        //异常处理或错误提示
        throw  new RuntimeException("叶子构件不能删除成员");
    }

    public void operation() {
        System.out.println("i'm leaf “”" + flag + " !");
    }
}
```

**示例**

```java
package cn.secretseater.designpattern.compositepattern;

public class CompositePattern {
    public static void main(String[] args) {
        Component root = initComponent();
        System.out.println("-----------根容器operation-----------");
        root.operation();
        System.out.println("-----------根-左容器operation-----------");
        root.getChild(0).operation();
        System.out.println("-----------根-左-叶子节点operation-----------");
        root.getChild(1).getChild(1).operation();
        System.out.println("-----------根-左-叶子节点add-----------");
        root.getChild(0).getChild(0).add(null);
    }

    public static Component initComponent(){
        // 创建根节点
        Component root = new Composite();
        // 在根节点下创建两个容器
        Component left = new Composite();
        Component right = new Composite();
        root.add(left);
        root.add(right);
        // 分别为左右节点创建两个叶子构件
        Component leaf1 = new Leaf();
        Component leaf2 = new Leaf();
        Component leaf3 = new Leaf();
        Component leaf4 = new Leaf();
        left.add(leaf1);
        left.add(leaf2);
        right.add(leaf3);
        right.add(leaf4);
        return root;
    }
}
```
```
# 运行结果：
-----------根容器operation-----------
i'm leaf 1 !
i'm leaf 7 !
i'm leaf 4 !
i'm leaf 0 !
-----------根-左容器operation-----------
i'm leaf 1 !
i'm leaf 7 !
-----------根-左-叶子节点operation-----------
i'm leaf 0 !
-----------根-左-叶子节点add-----------
Exception in thread "main" java.lang.RuntimeException: 叶子构件不能添加成员
	at cn.secretseater.designpattern.compositepattern.Leaf.add(Leaf.java:12)
	at cn.secretseater.designpattern.compositepattern.CompositePattern.main(CompositePattern.java:13)
```

## 透明组合模式

在透明组合模式 中，抽象构件Component中声明了所有用于管理成员对象的方法，包括`add()`、`remove(`)以及`getChild()`等方法，这样做的好处是确保所有构件类都有相同的接口。在客户端看来，叶子对象与容器对象所提供的方法是一致的，客户端可以一致地对待所有的对象。

透明组合模式的缺点是不够安全，因为叶子对象和容器对象在本质上是有区别的。也在叶子对象不可能有下一层次的对象，即不可能包含成员对象，因此为其提供`add()`、`remove(`)以及`getChild()`等方法是没有意义的，这在编译时不会出错，但在运行阶段如果调用这些方法可能会出错（如果没有提供相应的错误处理代码）。

## 安全组合模式

在安全组合模式中，抽象构件（Component）中没有生命任何用于管理成员对象的方法，而是在Composite中声明并实现这些方法。这种做法是安全的，因为根本不向叶子对象提供这些管理成员对象的方法，对于叶子对象，客户端不可能调用到这些方法。

安全组合模式的缺点是不够透明，因为叶子构件和容器构件具有不同的方法，且容器构件中那些用于管理成员对象的方法没有在抽象构件类中定义，因此客户端不能完全针对抽象编程，必须有区别地对待叶子构件和容器构件。在实际应用中，安全组合模式的使用频率也非常高。

## 优/缺点及适用环境

### 优点

- 可以清楚地定义分层次的复杂对象，表示对象的全部或部分层次，它让客户端忽略了层次的差异，方便对整个层次结构进行控制。
- 客户端可以一致地使用一个组合结构或其中单个对象，不必关心处理的是单个对象还是整个组合结构，简化了客户端代码。
- 在组合模式中增加新的容器构件和叶子构件都很方便，无须对现有类库进行任何修改，符合开闭原则。
- 为树形结构的面向对象实现提供了一种灵活的解决方案，通过叶子对象和容器对象的递归组合可以形成复杂的树形结构，但对树形结构的控制却非常简单。

### 缺点

- 在增加新构件时很难对容器中的构件类型进行限制。有时候希望一个容器中只能有某些特定类型的对象，例如某个文件夹只能包含文本文件，在使用组合模式时不能依赖类型系统来施加这些约束，因为它们都来自于相同的抽象层，在这种情况下必须通过在运行时进行类型检查来实现，这个实现过程较为复杂。

### 适用环境

- 在具有整体和部分的层次结构中希望通过一种忽略整体与部分的差异，客户端可以一致地对待它们。
- 在一个使用面向对象语言开发的系统中需要处理一个树形结构。
- 在一个系统中能够分离出叶子对象和容器对象，而且它们的类型不固定，需要增加一些新的类型。

