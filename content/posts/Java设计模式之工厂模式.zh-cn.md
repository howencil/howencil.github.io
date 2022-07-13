---
title: "Java设计模式之工厂模式"
date: 2022-07-13T10:27:32+08:00
draft: false
author: 插画师
tags: ["Java","设计模式"]
categories: ["Tech"]
---

# 1.工厂模式的3种形态

1. 简单工厂模式：又称为静态工厂方法模式；
2. 工厂方法模式：又称为多态性工厂模式；
3. 抽象工厂模式：又称为工具箱模式。



# 2.简单工厂模式

### 涉及的角色：

1. 工厂类角色：担任这个角色的是工厂方法模式的核心，含有与应用密切相关的商业逻辑，工厂类在客户端的直接调用下创建产品对象，由一个Java具体类实现；
2. 抽象产品角色：担任这个角色的类是由工厂方法模式所创建的对象的父类，或者说它们共同拥有的接口；
3. 具体产品角色：工厂方法模式所创建的任何对象都是这个角色的实例。

### 实现

```Java
public class Creator{
  // 静态工厂方法
  public static Product factory(){
    return new ConcreteProducr();
  }
}
```



```Java
public interface Product{}
```



```Java
public class ConcreteProduct implements Product{}
```

### 优点

1. 一个调用者想创建一个对象，只要知道起名称就可以了；
2. 扩展性高，如果想增加一个产品，只要扩展一个工厂类就可以；
3. 屏蔽产品的具体实现，调用者只关心产品的接口。

### 缺点

每次增加一个产品时，都需要增加一个具体的类和对象实现工具，使得系统中类的个数成倍增加，在一定程度上增加了系统的复杂度，同时也增加了系统具体类的依赖。



# 3.工厂方法模式

### 涉及的角色：

1. 抽象工厂角色：担任这个角色的是工厂方法模式的核心，任何在模式中创建对象的工厂类必须实现这个接口；
2. 具体工厂角色：担任这个角色的是实现抽象工厂接口的具体Java类，具体工厂角色含有和应用密切相关的逻辑，并且受到应用程序的调用来创建产品对象；
3. 抽象产品角色：工厂方法模式所创建的对象的超类型，产品对象共同的父类，或者说共同拥有的接口；
4. 具体产品角色：这个角色实现了抽象产品角色所声明的接口，工厂方法模式所创建的每一个对象都是某个具体产品角色的实例。

### 实现

```Java
public interface Creator{
  // 工厂方法
  Product factory();
}
```



```Java
public class ConcreteCreator1 implements Creator{
  @override
  public Product factory(){
    return new ConctereProduct1();
  }
}
```



```Java
public class ConcreteCreator2 implements Creator{
  @override
  public Product factory(){
    return new ConctereProduct2();
  }
}
```



```Java
public interface Product{}
```



```Java
public class ConctereProduct1 implements Product{}
```



```Java
public class ConctereProduct2 implements Product{}
```



# 4. 抽象工厂模式

### 涉及的角色：

1. 抽象工厂角色：担任这个角色的是工厂方法模式的核心，所有具体工厂类必须实现这个Java接口或者继承这个抽象的Java类；
2. 具体工厂角色：这个角色直接在客户端的调用下创建产品的实例，这个角色含有选择合适的产品对象的逻辑；
3. 抽象产品角色：担任这个角色的类是工作方法模式所创建的对象的父类，或它们共同拥有的接口；
4. 具体的产品角色：抽奖工厂模式所创建的每一个对象都是某个具体产品角色的实例。



### 实现

```Java
public interface Creator{
  // 产品A的工厂方法
  ProductA factoryA();
  
  //  产品B的工厂方法
  ProductB factoryB();
}
```



```Java
public class ConctereCreator1 implements Creator{
  @override
  public ProductA factoryA(){
    return new ProductA1();
  }
  
  @override
  public ProductB factoryB(){
    return new ProductB1();
  }
}
```



```Java
public class ConcreteCreator2 implements Creator{
  @override
  public ProductA factoryA(){
    return new ProductA2();
  }
  
  @override
  public ProductB factoryB(){
    return new ProductB2();
  }
}
```



```Java
public interface ProductA{}
```



```Java
public class ProductA1 implements ProductA{}
```



```Java
public class ProductA2 implements ProductA{}
```



```Java
public interface ProductB{}
```



```Java
public class ProductB1 implements ProductB{}
```



```Java
public class ProductB2 implements ProductB{}
```



> 参考文章：[女娲造人引发思考之Java设计模式：工厂模式](https://juejin.cn/post/6864562241199751182#heading-8)
