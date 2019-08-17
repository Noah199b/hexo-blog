---
title: Java中浅克隆与深克隆
date: 2019-08-17 12:34:29
tags:
	- Java
	- 设计模式
	- 浅克隆与深克隆
categories:
	- Java设计模式
---

## 浅克隆（Shallow Clone）

在浅克隆中，如果原型对象的成员变量是值类型（如int，double，byte，boolean，char等基本数据类型），将复制一份给克隆对象；如果原型对象的成员变量是引用类型（如类，接口，数组等复杂数据类型），则将引用对象的地址复制一份给克隆对象，也就是说原型对象和克隆对象的成员变量指向相同的内存地址。

## 深克隆（Deep Clone）

在深克隆中，无论原型对象的成员变量是值类型还是引用类型，都将复制一份给克隆对象，深克隆将原型对象的所有引用对象也复制一份给克隆对象。

## Java中clone()方法和Cloneable接口

在Java中，所有的Java类均继承自`java.lang.Object()`类，Object类提供了一个`clone()`方法，可以将一个Java对象复制一份。因此在Java中可以直接使用Object提供的`clone()`方法来实现浅克隆。

需要注意的是能够实现克隆的Java类必须实现一个标识接口Cloneable，表示这个Java类支持被复制。如果一个类没有实现这个接口但是调用了`clone()`方法，Java编译器将抛出一个`CloneNotSupportedException`异常。

<!-- more-->

**错误示例：**

```java
package cn.secretseater.clone;

public class Person {
    private String name;
    private Gender gender;

    //省略Getter和Setter
    //省略toString()方法

    protected Person clone() throws CloneNotSupportedException {
        return (Person) super.clone();
    }
}

```
```java
package cn.secretseater.clone;

public class ShallowClone {
    public static void main(String[] args) {
        Gender gender = new Gender();
        gender.setGender('m');
        Person person = new Person();
        person.setName("Tom");
        person.setGender(gender);

        try {
            Person newPerson = person.clone();
        } catch (CloneNotSupportedException e) {
            e.printStackTrace();
        }
    }
}
```
```
# 运行结果：
java.lang.CloneNotSupportedException: cn.secretseater.clone.Person
	at java.lang.Object.clone(Native Method)
	at cn.secretseater.clone.Person.clone(Person.java:32)
	at cn.secretseater.clone.ShallowClone.main(ShallowClone.java:12)
```

Java语言中的`clone()`方法满足一下几点：

- 对任何对象 `x`，都有 `x.clone!=x`,即克隆对象与原型对象不是同一个。
- 对任何对象 `x`，都有 `x.clone.getClass() == x.getClass()`，即克隆对象与原对象的类型一样。
- 如果对象 `x`的`equals()`方法定义恰当，那么 `x.clone.equals(x)`应该成立。

为了获取对象的一个克隆，可以直接利用Object类的`clone()`方法，具体步骤如下：

- 在派生类中覆盖基类的`cloen()`方法，并声明为public。
- 在派生类的`clone()`方法中调用`super.clone()`。
- 派生类需实现Cloneable接口。

### 深克隆解决方案

**通过序列化操作实现深克隆**

```java
package cn.secretseater.clone;

import java.io.*;

public class Person implements Serializable{
    private String name;
    private Gender gender;

    //省略Getter和Setter
    //省略toString()方法

    public Person deepClone() throws IOException,ClassNotFoundException{
        // 将对象写入流中
        ByteArrayOutputStream bos=new ByteArrayOutputStream();
        ObjectOutputStream oos= null;
        oos.writeObject(this);
        oos.flush();
        //将对象从流中取出
        ByteArrayInputStream bis=new ByteArrayInputStream(bos.toByteArray());
        ObjectInputStream ois=new ObjectInputStream(bis);
        return (Person)ois.readObject();
    }
}
```
```java
import java.io.Serializable;

public class Gender implements Serializable {
   private  char gender;

   //省略Getter和Setter
   //省略toString()方法
}
```
```
# 运行结果：
-------------原型对象-------------
Person{name='Tom', gender=Gender{gender=m}}
-------------克隆对象-------------
Person{name='Tom', gender=Gender{gender=m}}
-------------修改引用-------------
修改后......
-------------原型对象-------------
Person{name='Tom', gender=Gender{gender=m}}
-------------克隆对象-------------
Person{name='Jerry', gender=Gender{gender=f}}
```