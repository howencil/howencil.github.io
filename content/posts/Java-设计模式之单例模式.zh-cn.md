---
title: "Java 设计模式之单例模式"
date: 2022-07-12T10:52:37+08:00
draft: false
author: ["插画师"]
tag: ["Java","设计模式"]
categories：["Tech"]
---

# 1.单例模式的特点

1. 单例类只有一个实例对象；
2. 该单例对象必须由单例类自行创建；
3. 单例类对外提供一个访问该单例的全局访问点。

# 2.单例模式的优缺点

#### 1.优点：

1. 单例模式可以保证内存里只有一个实例，减少了内存的开销；
2. 可以避免对资源的多重占用；
3. 单例模式设置全局访问点，可以优化和共享资源的访问。

#### 2.缺点

1. 单例模式一般没有接口，扩展困难。如果要扩展，则除了修改原来的代码，没有第二种途径，违背开闭原则；
2. 在并发测试中，单例模式不利于代码调试。在调试过程中，如果单例中的代码没有执行完，也不能模拟生成一个新的对象。
3. 单例模式的功能代码通常写在一个类中，如果功能设计不合理，则很容易违背单一职责原则。

# 3.单例模式的应用场景

1. 需要频繁创建的一些类，使用单例可以降低系统的内存压力，减少GC；
2. 某些类只要求生成一个对象时；
3. 某些类创建实例时占有资源较多，或实例化耗时较长，且经常使用；
4. 某些类需要频繁实例化，而创建的对象又频繁被销毁时，如多线程的线程池、网络连接池等；
5. 频繁访问数据库或文件的对象；
6. 对于一些控制硬件级别的操作，或者从系统上来讲应当是单一控制逻辑的操作，如果有多个实例，则系统会完全乱套；
7. 当对象需要被共享的场合。由于单例模式只允许创建一个对象，共享该对象可以节省内存，并加快对象访问速度。如Web中的配置对象、数据库的连接池等。

# 4.单例模式的实现

#### 1.懒汉式

```Java
public class LazySingleton{
  private static volatile LazySingleton instance = null;
  private LazySingleton(){}
  // 如果没有synchronized就是线程不安全的懒汉式
  private static synchronized LazySingleton getInstance(){
    if(instance == null){
      instance = new LazySingleton();
    }
   	return instance;
  }
}
```



#### 2.饿汉式

```Java
public class Singleton{
  private static volatile Singleton instance = new Singleton();
  private Singleton(){}
  private static Singleton getInstance(){
    return instance;
  }
}
```



#### 3.双重校验锁 - 线程安全

```Java
public class Singleton{
  private static volatile Singtelon instance;
  private Singleton(){}
  public static Singleton getInstance(){
    if(instance == null){
      synchronized(Singleton.class){
        if(instance == null){
          instance = new Singleton();
        }
      }
    }
    return instance;
  }
}
```

#### 4.静态内部类实现

```Java
public class Singleton{
  private Singletion(){}
  private static class SingletonHolder{
    private static final Singleton INSTANCE = new Singleton();
  }
  public static Singleton getInstance(){
    return SingletonHolder.INSTANCE;
  }
}
```

#### 5.枚举实现

```Java
public enum Singleton{
  INSTANCE;
}
```

