---
title: 外观模式（Facade Pattern）
date: 2019-09-04 21:28:14
tags:
	- Java
	- 设计模式
	- 外观模式
categories:
	- Java设计模式
---

 > 参考书籍：《Java设计模式》 @刘伟

 在软件开发中有时候为了完成一项较为复杂的功能，一个客户类需要和多个业务类交互，而这些需要交互的业务类经常会作为一个整体出现，由于涉及的类较多，导致使用时代码较为复杂，此时特别需要一个类似服务员的角色，由它来负责和多个业务类进行交互，而客户类只需要与该类交互。

 外观模式通过引入一个新的外观（Facade）来实现该功能，外观类充当了软件系统中“服务员”，它为多个业务类的调用提供一个统一的入口，简化了类与类之间的交互。在外观模式中，那些需要交互的业务类被称为子系统（Subsystem）。

 **外观模式：为子系统中的一组接口提供一个统一的入口。外观模式定义了一个高层接口，这个接口使得这一子系统更加容易使用。**

 ## 结构

 - **Facade（外观角色）**：在客户端可以调用它的方法，在外观角色中可以知道相关的（一个或多个）子系统的功能和责任；在正常情况下，它将所有从客户端发来的请求委派到相应的子系统，传递给相应的子系统对象处理。
 - **SubSystem（子系统角色）**：在软件系统中可以有一个或多个子系统角色，每一个子系统可以不是一个单独的类，而是一个类的集合，它实现子系统的功能；每一个子系统都可以被客户端直接调用，或者被外观角色调用，它处理外观类传来的请求；子系统并不知道外观的存在，对于子系统而言，外观角色仅仅是另一个客户端而已。

 ## 实现

<!-- more-->

**子系统**

```java
package cn.secretseater.designpattern.facadepattern;

import java.io.FileInputStream;
import java.io.FileNotFoundException;
import java.io.IOException;

/**
 * 文件读取，充当子系统
 */
public class FileReader {
    public String read(String fileNamsSrc){
        System.out.print("读取文件，获取明文：");;
        StringBuilder sb = new StringBuilder();
        FileInputStream fis = null;
        try {
            fis = new FileInputStream(fileNamsSrc);
            int data;
            while((data = fis.read())!= -1){
                sb = sb.append((char)data);
            }
        } catch (FileNotFoundException e) {
            e.printStackTrace();
        } catch (IOException e) {
            e.printStackTrace();
        } finally {
            if (fis != null) {
                try {
                    fis.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
        }
        System.out.println(sb.toString());;
        return sb.toString();
    }
}

```
```java
package cn.secretseater.designpattern.facadepattern;

/**
 * 数据加密类，充当子系统类
 */
public class CipherMachine {
    public String encrypt(String plainText){
        System.out.print("数据加密，将明文转换为密文：");
        String es = "";
        for(char ch:plainText.toCharArray()){
            es += String.valueOf(ch % 7);
        }
        System.out.println(es);
        return es;
    }
}
```
```java
package cn.secretseater.designpattern.facadepattern;

import java.io.FileNotFoundException;
import java.io.FileOutputStream;
import java.io.IOException;

/**
 * 文档保存类，充当子系统类
 */
public class FileWriter {
    public void write(String encryptStr,String fileName){
        System.out.println("保存密文，写入文件！");
        FileOutputStream fos = null;
        try {
            fos = new FileOutputStream(fileName);
            fos.write(encryptStr.getBytes());
        } catch (FileNotFoundException e) {
            e.printStackTrace();
        } catch (IOException e) {
            e.printStackTrace();
        }finally {
            if (fos != null) {
                try {
                    fos.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
        }
    }
}
```

**外观角色**

```java
package cn.secretseater.designpattern.facadepattern;

public class EncryptFacade {
    //维持子系统对象的引用
    private FileReader fileReader = new FileReader();
    private CipherMachine cipherMachine = new CipherMachine();
    private FileWriter fileWriter = new FileWriter();

    public EncryptFacade() {
        fileReader = new FileReader();
        cipherMachine = new CipherMachine();
        fileWriter = new FileWriter();
    }
    //调用子系统对象的业务方法
    public void fileEncrypt(String fileNameSrc,String fileNameDes){
        String plainStr = fileReader.read(fileNameSrc);
        String encryptStr = cipherMachine.encrypt(plainStr);
        fileWriter.write(encryptStr,fileNameDes);
    }
}
```

**示例**

```java
package cn.secretseater.designpattern.facadepattern;

public class FacadePattern {
    public static void main(String[] args) {
        EncryptFacade encryptFacade = new EncryptFacade();
        encryptFacade.fileEncrypt("src/main/java/cn/secretseater/designpattern/facadepattern/src.txt",
                "src/main/java/cn/secretseater/designpattern/facadepattern/des.txt");
    }
}
```
```
# 运行结果：
读取文件，获取明文：i'm FacadePattern !
数据加密，将明文转换为密文：0444061623364432545
保存密文，写入文件！
```

## 抽象外观类

在标准的外观结构图中，如果需要增加、删除或更换与外观类交互的子系统类，必须修改外观类或客户端的源码，这将违背开闭原则，因此可以通过引入抽象外观类对系统进行改进，这在一定程度上可以解决该问题。

**新增抽象外观**

```java
package cn.secretseater.designpattern.facadepattern;

public abstract class EncryptFacade {
    public abstract void fileEncrypt(String fileNameSrc,String fileNameDes);
}
```

**外观实现类**

```java
package cn.secretseater.designpattern.facadepattern;

public class EncryptFacadeImpl extends EncryptFacade {
    //维持子系统对象的引用
    private FileReader fileReader = new FileReader();
    private CipherMachine cipherMachine = new CipherMachine();
    private FileWriter fileWriter = new FileWriter();

    public EncryptFacadeImpl() {
        fileReader = new FileReader();
        cipherMachine = new CipherMachine();
        fileWriter = new FileWriter();
    }
    //调用子系统对象的业务方法
    public void fileEncrypt(String fileNameSrc,String fileNameDes){
        String plainStr = fileReader.read(fileNameSrc);
        String encryptStr = cipherMachine.encrypt(plainStr);
        fileWriter.write(encryptStr,fileNameDes);
    }
}
```

通过反射生成对象，新增外观只需修改配置文件，无须修改源代码，符合开闭原则。

## 优/缺点及适用环境

### 优点

- 它对客户端屏蔽了子系统组件，减少了客户端所需处理的对象数目，并使子系统使用起来更加容易。通过引入外观模式，客户端代码将变得简单，与之关联的对象也很少。
- 它实现了子系统与客户端之间的松耦合关系，这使得子系统的变化不会影响到调用它的客户端，只需要调整外观类即可。
- 一个子系统的修改对其他子系统没有任何影响，而且子系统内部变化也不会影响到外观对象。

### 缺点

- 不能很好地限制客户端直接使用子系统类，如果对客户端访问子系统类做太多的限制则减少了可变性和灵活性。
- 如果涉及不当，增加新的系统可能需要修改外观类的源代码，违背了开闭原则。

### 适用环境

- 当要访问一系列复杂的子系统提供一个简单入口时，可以使用外观模式。
- 客户端程序与多个子系统之间存在很大的依赖性。引入外观类可以将子系统与客户端解耦，从而提高子系统的独立性和可移植性。
- 在层次化结构中可以是外观模式定义系统中每一层的入口，层与层之间不直接产生联系，而通过外观类建立联系，降低层之间的耦合度。