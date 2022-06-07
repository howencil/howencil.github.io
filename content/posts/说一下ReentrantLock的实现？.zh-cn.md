---
title: "说一下ReentrantLock的实现？"
date: 2022-06-07T15:16:02+08:00
draft: false
author: 插画师
tags: ["Java"]
categories: ["Tech"]
---

## 1.什么是ReentrantLock
**ReentrantLock是一个可重入的互斥锁。**
1. *可重入性*：可以支持一个线程对锁的重复获取
2. *公平锁/非公平锁*：==公平锁==，顾名思义就是获取策略相对公平，多个线程在获取同一个锁时，按照锁的申请时间来依次获得锁，不能插队。==非公平锁==，就是当锁释放时，等待中的线程均有机会获得锁，ReentrantLock默认就是非公平的，但可以通过带boolean参数的构造参数来指定使用公平锁。==一般来说，非公平锁的性能要由于公平锁。==

## 2.源码解读
- **ReentrantLock是基于AQS的，AQS是Java并发包中众多同步组件的构建基础，它通过一个int类型的状态变量state和一个FIFO队列来完成共享资源的获取，线程的排队等。AQS是个底层框架，采用模板方法模式，他定义了通用的较为复杂的逻辑骨架，比如线程的排队、阻塞、唤醒等，将这些复杂但实质通用的部分抽取出来，这些都是需要构建同步组件的使用者无需关心的，使用者仅需重写一些简单的指定的方法即可（其实就是对于共享变量state的一些简单的获取释放操作）。**

###### 1.常用方法
1. 无参构造器（默认是非公平锁）
```Java
public ReentrantLock() {
        sync = new NonfairSync();//默认是非公平的
    }
```
2. 带布尔值的构造器（是否公平）
```Java
public ReentrantLock(boolean fair) {
        sync = fair ? new FairSync() : new NonfairSync();//fair为true，公平锁；反之，非公平锁
    }
```
3. lock()
```Java
public void lock() {
        sync.lock();//代理到Sync的lock方法上
    }
```
4. lockInterruptibly()
```Java
public void lockInterruptibly() throws InterruptedException {
        sync.acquireInterruptibly(1);//代理到sync的相应方法上，同lock方法的区别是此方法响应中断
    }
```
5. tryLock()
```Java
public boolean tryLock() {
        return sync.nonfairTryAcquire(1);//代理到sync的相应方法上
    }
```
6. unlock()
```Java
public void unlock() {
        sync.release(1);//释放锁
    }
```
7. newCondition()
```Java
public void unlock() {
        sync.release(1);//释放锁
    }
```
- 小结： **其内部定义了3个重要的静态内部类，Sync、NonFairSync、FairSync。Sync作为ReentrantLock中公用的同步组件，继承了AQS（作用是线程排队、阻塞、唤醒等）；NonFairSync、FairSync两者都继承了Sync，调用了Sync的公用逻辑，然后在各自的内部完成自己的特定逻辑（公平或非公平）。**

###### 2.NonFairSync（非公平可重入锁）
- NonFairSync
```Java
static final class NonfairSync extends Sync {//继承Sync
        private static final long serialVersionUID = 7316153563782823691L;
        /** 获取锁 */
        final void lock() {
            if (compareAndSetState(0, 1))//CAS设置state状态，若原值是0，将其置为1
                setExclusiveOwnerThread(Thread.currentThread());//将当前线程标记为已持有锁
            else
                acquire(1);//若设置失败，调用AQS的acquire方法，acquire又会调用我们下面重写的tryAcquire方法。这里说的调用失败有两种情况：1当前没有线程获取到资源，state为0，但是将state由0设置为1的时候，其他线程抢占资源，将state修改了，导致了CAS失败；2 state原本就不为0，也就是已经有线程获取到资源了，有可能是别的线程获取到资源，也有可能是当前线程获取的，这时线程又重复去获取，所以去tryAcquire中的nonfairTryAcquire我们应该就能看到可重入的实现逻辑了。
        }
        protected final boolean tryAcquire(int acquires) {
            return nonfairTryAcquire(acquires);//调用Sync中的方法
        }
    }
```
- nonfairTryAcquie()
```Java
final boolean nonfairTryAcquire(int acquires) {
            final Thread current = Thread.currentThread();//获取当前线程
            int c = getState();//获取当前state值
            if (c == 0) {//若state为0，意味着没有线程获取到资源，CAS将state设置为1，并将当前线程标记我获取到排他锁的线程，返回true
                if (compareAndSetState(0, acquires)) {
                    setExclusiveOwnerThread(current);
                    return true;
                }
            }
            else if (current == getExclusiveOwnerThread()) {//若state不为0，但是持有锁的线程是当前线程
                int nextc = c + acquires;//state累加1
                if (nextc < 0) // int类型溢出了
                    throw new Error("Maximum lock count exceeded");
                setState(nextc);//设置state，此时state大于1，代表着一个线程多次获锁，state的值即是线程重入的次数
                return true;//返回true，获取锁成功
            }
            return false;//获取锁失败了
        }
```
- 总结流程：
	- 1.*先获取state值，若为0，意味着此时没有线程获取资源，CAS将其设置为1，设置成功则代表获取到排他锁了；*
	- 2.*若state大于0，肯定有线程已经抢占到资源了，此时再去判断是否就是自己抢占的，是的话，state累加，返回true，重入成功，state的值即是线程重入的次数。*
	- 3.*其他情况，则获取锁失败*

###### 3.FairSync
- FairSync
```Java
static final class FairSync extends Sync {
        private static final long serialVersionUID = -3000897897090466540L;

        final void lock() {
            acquire(1);//直接调用AQS的模板方法acquire，acquire会调用下面我们重写的这个tryAcquire
        }

        protected final boolean tryAcquire(int acquires) {
            final Thread current = Thread.currentThread();//获取当前线程
            int c = getState();//获取state值
            if (c == 0) {//若state为0，意味着当前没有线程获取到资源，那就可以直接获取资源了吗？NO!这不就跟之前的非公平锁的逻辑一样了嘛。看下面的逻辑
                if (!hasQueuedPredecessors() &&//判断在时间顺序上，是否有申请锁排在自己之前的线程，若没有，才能去获取，CAS设置state，并标记当前线程为持有排他锁的线程；反之，不能获取！这即是公平的处理方式。
                    compareAndSetState(0, acquires)) {
                    setExclusiveOwnerThread(current);
                    return true;
                }
            }
            else if (current == getExclusiveOwnerThread()) {//重入的处理逻辑，与上文一致，不再赘述
                int nextc = c + acquires;
                if (nextc < 0)
                    throw new Error("Maximum lock count exceeded");
                setState(nextc);
                return true;
            }
            return false;
        }
    }
```
 - 总结：*公平锁与非公平锁的逻辑基本一致，不同的地方在于hasQueuedPredecessors()这个逻辑判断。即使state为0，也不会直接去获取资源，要先判断存在还在排队的现场，如果不存在，才会去尝试获取，做后面的处理。反之，返回false，获取失败。*
- hasQueuedPredecessors()
```Java
public final boolean hasQueuedPredecessors() {
        Node t = tail; // 尾结点
        Node h = head;//头结点
        Node s;
        return h != t &&
            ((s = h.next) == null || s.thread != Thread.currentThread());//判断是否有排在自己之前的线程
    }
```
**需要注意的是，这个判断是否有排在自己之前的线程的逻辑稍微有些绕，我们来梳理下，由代码得知，有两种情况会返回true，我们将此逻辑分解一下（注意：返回true意味着有其他线程申请锁比自己早，需要放弃抢占）**

  1. **h !=t && (s = h.next) == null，这个逻辑成立的一种可能是head指向头结点，tail此时还为null。考虑这种情况：当其他某个线程去获取锁失败，需构造一个结点加入同步队列中（假设此时同步队列为空），在添加的时候，需要先创建一个无意义傀儡头结点（在AQS的enq方法中，这是个自旋CAS操作），有可能在将head指向此傀儡结点完毕之后，还未将tail指向此结点。很明显，此线程时间上优于当前线程，所以，返回true，表示有等待中的线程且比自己来的还早。**

       

  2. **h != t && (s = h.next) != null && s.thread != Thread.currentThread()。同步队列中已经有若干排队线程且当前线程不是队列的老二结点，此种情况会返回true。假如没有s.thread !=Thread.currentThread()这个判断的话，会怎么样呢？若当前线程已经在同步队列中是老二结点（头结点此时是个无意义的傀儡结点),此时持有锁的线程释放了资源，唤醒老二结点线程，老二结点线程重新tryAcquire（此逻辑在AQS中的acquireQueued方法中），又会调用到hasQueuedPredecessors，不加s.thread !=Thread.currentThread()这个判断的话，返回值就为true，导致tryAcquire失败。**
       *最后，来看看ReentrantLock的tryRelease，定义在Sync中*
  ```Java
protected final boolean tryRelease(int releases) {
            int c = getState() - releases;//减去1个资源
            if (Thread.currentThread() != getExclusiveOwnerThread())
                throw new IllegalMonitorStateException();
            boolean free = false;
            //若state值为0，表示当前线程已完全释放干净，返回true，上层的AQS会意识到资源已空出。若不为0，则表示线程还占有资源，只不过将此次重入的资源的释放了而已，返回false。
            if (c == 0) {
                free = true;//
                setExclusiveOwnerThread(null);
            }
            setState(c);
            return free;
        }
  ```
###### 4.全文总结
**ReentrantLock是一种可重入的，可实现公平性的互斥锁，它的设计基于AQS框架，可重入和公平性的实现逻辑都不难理解。每重入一次，state就加1，当然在释放的时候，也得一层一层地释放。至于公平性，在尝试获取锁的时候多了一个判断：是否有比自己申请早的线程在同步队列中等待，如果有，去等待；如果没有，才允许去抢占。**

> [ReentrantLock实现原理及源码分析 - dreamcatcher-cx - 博客园](https://www.cnblogs.com/chengxiao/p/7255941.html)
