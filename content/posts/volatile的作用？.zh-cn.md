---
title: "Volatile的作用？"
date: 2022-06-07T15:19:40+08:00
draft: false
author: 插画师
tags: ["Java"]
categories: ["Tech"]
---

## 1.volatile的作用
1. 能保证共享变量的可见性，即一个线程对共享变量进行修改，其他线程能够立即获得修改后的值。（*volatile变量修改后，JVM会立即将缓存中的值写会主内存，也就是刷新*）。
2. 禁止操作的指令重排，保证操作的有序性。

**PS：Java内存模型规定了所有的变量都存储在主内存中，线程的工作内存中保存了该线程使用的变量副本（从主内存中拷贝的），线程对变量的所有操作（读取、赋值等）都必须在工作线程中进行，不能直接读写主内存的数据。线程间变量值的传递只能通过主内存完成。**
## 2.volatile应用场景必须满足的条件（多线程中）
1. 运算结果不依赖变量的值；
2. 变量不需要与其他的转态变量共同参与不变的约束。

**PS：1.第一个条件的理解：volatile变量不能是自增量（如x++)，x++实际上是3步运算，-1获得x值，-2修改x值，-3将x值写回内存；
2.第二个条件的理解：不变的约束即为条件表达式（start<end)，如果表达式使用了两个及以上的变量（不管是否是volatile的变量）就无法保证正确性，也就是说不变式会失效。**
## 3.应用场景举例分析
1.状态指标
*volatile变量用于状态标志，用于指示一个重要的一次性事件，如初始化完成或请求停机。*
```Java
volatile boolean shutdownRequested;  
public void shutdown() {   
    shutdownRequested = true;   
}  
public void doWork() {   
    while (!shutdownRequested) {   
        // 代码业务逻辑  
    }  
}
```
**分析：**线程A执行doWork()的逻辑代码时，线程B调用了shutdown()方法，线程A立即停止运行`while`中的代码。
2.一次性发布
*在缺乏同步的情况下，可能遇到某个对象引用的更新值（由另一个线程写入）和该对象状态的就值同事存在。例如双重检验单例DCL。*
```Java
public class Singleton{
    private volatile static Singleton uniqueInstance;
    private Singleton(){
    }
    public static Singleton getUniqueInstance(){
        if(uniqueInstance == null){                   // 1)
            synchronized(Singleton.class){            
                if(uniqueInstance == null){           // 2)
                    uniqueInstance = new Singleton(); // 3)
                }
            }
        }
        return uniqueInstance;
    }
    public static void main(String[] args){
        Singleton.getUniqueInstance();
    }
}
```
**对象的创建过程：类加载检查 -> 为对象分配内存 -> 将分配的内存空间（不包括对象头）初始化置零 -> 设置对象头信息 -> 执行`init()`**
按照用户定义的构造函数进行初始化。

**分析：线程A运行到图中2的地方获得同步锁，`uniqueInstance==null`运行图中3的地方创建对象，JVM分3步（实际上是5步）运行的，1.为`uniqueInstance`分配内存空间；2.初始化`uniqueInstance`；3.将`uniqueInstance`指向分配好的内存空间。如果没有用`volatile`修饰，JVM进行了指令重排，执行了1->3步，但是没有执行2的初始化，此时线程B执行`getUniqueInstance()`发现`uniqueInstance`不为null，便返回还没有初始化过的对象，就会出现错误。即线程A执行3时1->3状态下未初始化的对象（旧值）和执行完3的1->3->2状态下初始化后的对象（新值）同时存在。**

## 4.开销较低的“读写锁”
如果读操作远远超过了写操作，可以结合内部锁和`volatile`变量来减少公共代码路径的开销。例如线程安全的计数器，使用`synchronized`确保增量操作的原子性，并使用`volatile`保证当前结果的可见性。如果更新不频繁，该方法可以实现更好的性能。因为读开销仅仅涉及`volatile`操作，优于一个无竞争的锁的开销。
```Java
public class CheesyCounter {  
    private volatile int value;  
    //读操作，没有synchronized，提高性能  
    public int getValue() {   
        return value;   
    }   
    //写操作，必须synchronized。因为x++不是原子操作  
    public synchronized int increment() {  
        return value++;  
    }  
}
```
## 5."volatile bean"模式
"volatile bean"模式的基本原理：*很多框架为易变数据的持有者（HttpSession）提供容器，放入这些容器的对线必须是线程安全的。在"volatile bean"模式中，JavaBean的所有成员都是`volatile`类型的，并且`getter()`和`setter()`方法不能包含约束条件（即逻辑判断）。*
```Java
public class Person {  
    private volatile String firstName;  
    private volatile String lastName;  
    private volatile int age;
    
    public String getFirstName() { return firstName; }  
    public String getLastName() { return lastName; }  
    public int getAge() { return age; }  
    public void setFirstName(String firstName) {   
        this.firstName = firstName;  
    }  
    public void setLastName(String lastName) {   
        this.lastName = lastName;  
    }  
    public void setAge(int age) {   
        this.age = age;  
    }  
}
```
> [volatile关键字的作用及使用场景 | HBC'Blog](https://corbinhu.github.io/2020/04/15/use-volatile-scens/)
