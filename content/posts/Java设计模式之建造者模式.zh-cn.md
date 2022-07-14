---
title: "Java设计模式之建造者模式"
date: 2022-07-14T11:14:08+08:00
draft: false
author: 插画师
tags: ["Java","设计模式"]
categories: ["Tech"]
---

# 1.定义

建造者模式是对象的创建型模式，可以将一个产品的内部表象与产品的生成过程分割开来，从而使一个建造过程生成具有不同的内部表象的产品对象。



建造者模式将产品的结构和产品的零件建造过程对客户隐藏起来，把对建造过程进行指挥的责任和具体建造者零件的责任分割开来，达到责任划分和封装的目的。



# 2.意图

将一个复杂的构建与其表示相分离，使得同意的构建过程可以创建不同的表示。



# 3.主要解决问题

主要解决在软件系统中，有时候面临着“一个复杂对象”的创建工作，其通常由各个部分的子对象用一定的算法构成；由于需求的变化，这个复杂对象的各个部分经常面临着剧烈的变化，但是将它们组合在一起的算法却相对稳定。



# 4.何时使用

一些基本部件不会变，而其组合经常变化的时候。



# 5.优缺点

### 优点

1. 建造者独立，易扩展；
2. 便于控制细节风险。

### 缺点

1. 产品必须有共同点，范围有限制；
2. 如果内部变化复杂，会有很多的建造类。



# 6.结构

1. 抽象建造者角色：给出一个抽象接口，用来规范产品对象的各个组成成分的建造，一般来说，产品所包含的零件数目和建造方法的数目相等，换言之，有多少个零件，就有多少个相应的建造方法；
2. 具体建造者角色：实现抽象建造者所声明的接口，给出一步步的完成创建产品实例的操作；在建造过程完成后，提供产品的实例；
3. 导演者角色：担任这个角色的类调用具体建造者来创建产品对象，导演者角色并没有产品类的具体知识，真正拥有产品类的具体知识是具体建造者角色；
4. 产品角色：产品便是建造中的复杂对象，一般来说，一个系统中会有多于一个的产品类，而且这些产品类并不一定有共同的接口，而完全可以是不相关的。

一般来说，每有一个产品类，就有一个相应的具体建造者类，这些产品应当有一样数目的零件，而每有一个零件，就相应的在所有建造者角色中有一个建造方法。



# 7.实现

```Java
public interface Builder{
  void buildPart1();
  void buildPart2();
  Product retrieveResult();
}
```



```Java
public class ConcreteBuilder implements Builder{
  private Product product = new Product();
  @override
  public void buildPart1(){
    // 建造零件的实现
  }
  
  @override 
  public void buildPart2(){
    // 建造零件的实现
  }
  
  // 返回产品
 @override
  public Product retrieveResult(){
    return product;
  }
}
```



```Java
public class Directro{
  private Builder builder;
  /**
   * 产品构造方法，负责调用各个零件构造方法
   */
  public void construct(){
    builder = new ConcreteBuilder();
    builder.buildPart1();
    builder.buildPart2();
    builer.retrieveResult();
  }
}
```



```Java
public class Product{}
```



> 参考文章：[安装电脑思考到了Java设计模式：建造者模式](https://juejin.cn/post/6866754495091572743)
