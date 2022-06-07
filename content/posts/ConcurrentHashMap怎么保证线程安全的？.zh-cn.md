---
title: "ConcurrentHashMap怎么保证线程安全的？"
date: 2022-06-07T15:14:26+08:00
draft: false
author: 插画师
tags: ["Java"]
categories: ["Tech"]
---

## 1.先说结论
ConcurrentHashMap主要使用的是CAS+自旋+synchronized+多重check来保证在初始化、新增、扩容时数据安全，读取数据时使用volatile让元素节点在多线程之间可见，从而达到获取最新的值。
## 2.initTable初始化
```Java
    /**
     * Initializes table, using the size recorded in sizeCtl.
     */
    private final Node<K,V>[] initTable() {
        Node<K,V>[] tab; int sc;
        while ((tab = table) == null || tab.length == 0) {//自旋处理 保证了一定初始化成功
            if ((sc = sizeCtl) < 0)//小于0 说明 有线程正在进行初始化  就让出CPU资源
                Thread.yield(); // lost initialization race; just spin
            else if (U.compareAndSwapInt(this, SIZECTL, sc, -1)) {
                try {
                    if ((tab = table) == null || tab.length == 0) {
                        int n = (sc > 0) ? sc : DEFAULT_CAPACITY;
                        @SuppressWarnings("unchecked")
                        Node<K,V>[] nt = (Node<K,V>[])new Node<?,?>[n];
                        table = tab = nt;
                        sc = n - (n >>> 2);
                    }
                } finally {
                    sizeCtl = sc;
                }
                break;
            }
        }
        return tab;
    }
```
1. 自旋，保证一定初始化完成；
2. CAS保证同一时间只有一个初始化线程；
3. 二次check保证数据初始化时`table`是对的。

## 3.put的线程安全
```Java
    /** Implementation for put and putIfAbsent */
    final V putVal(K key, V value, boolean onlyIfAbsent) {
        if (key == null || value == null) throw new NullPointerException();
        int hash = spread(key.hashCode());//计算 得到哈希值
        int binCount = 0;
        for (Node<K,V>[] tab = table;;) {//这边也是一个自旋操作 保证一定能新增元素成功
            Node<K,V> f; int n, i, fh;
            if (tab == null || (n = tab.length) == 0)
                tab = initTable();//执行初始化操作
            else if ((f = tabAt(tab, i = (n - 1) & hash)) == null) {
             <!--这边也是一个CAS 操作 只有 的当前节点值是null 的时候才可以网卡槽里面放入元素-->
                if (casTabAt(tab, i, null,
                             new Node<K,V>(hash, key, value, null)))
                    break;                   // no lock when adding to empty bin
            }
            else if ((fh = f.hash) == MOVED)//MOVED 是一个固定的值 是为了标识 当前正在扩容，此节点已被转移，不能操作 需要等待扩容后 才能操作，此时是自悬等待
                tab = helpTransfer(tab, f);
            else {
                V oldVal = null;
                synchronized (f) { //这边使用了synchronized锁哈
                    //Hash冲突后处理
                }
                if (binCount != 0) {
                    if (binCount >= TREEIFY_THRESHOLD)//如果链表的长度大于等8 就要转变为红黑树
                        treeifyBin(tab, i);
                    if (oldVal != null)
                        return oldVal;
                    break;
                }
            }
        }
        addCount(1L, binCount);// 这边新增数量大小，然后判断是否要进行扩容
        return null;
    }
```
1. 自旋：自旋保证了元素一定能成功放入；
2. CAS：CAS操作保证了放入数据时当前节点一定是null的，如果在这期间有别的线程也来操作这个卡槽的值，那CAS中的`table`中的`i`位置的值就一定不是null了，和预期值不符，那就CAS执行失败，进入下次循环。在下次循环时，发现此槽位已经有值了，就会走到`Hash`碰撞后的`Synchronized`的方法块里，这样保证了在新的卡槽新增的第一个元素是线程安全的；
3. Synchronized锁：保证执行的线程安全。

## 4.transfer扩容时的线程安全
```Java
 /**
     * Moves and/or copies the nodes in each bin to new table. See
     * above for explanation.
     */
    private final void transfer(Node<K,V>[] tab, Node<K,V>[] nextTab) {
        int n = tab.length, stride;
        if ((stride = (NCPU > 1) ? (n >>> 3) / NCPU : n) < MIN_TRANSFER_STRIDE)
            stride = MIN_TRANSFER_STRIDE; // 这边是设置CPU 可操作数组的node步长 最低 是16
        <!--tab是新的数组  这边的nextTab 就是扩容后的数组 -->    
        if (nextTab == null) {            // initiating
            try {
                @SuppressWarnings("unchecked")
                Node<K,V>[] nt = (Node<K,V>[])new Node<?,?>[n << 1];//扩容的时候 是n << 1 结果相当于n*2
                nextTab = nt;
            } catch (Throwable ex) {      // try to cope with OOME
                sizeCtl = Integer.MAX_VALUE;
                return;
            }
            nextTable = nextTab;//当在扩容过程中的时候 一个过渡的表
            transferIndex = n;// 这边transferIndex 是扩容的时候 第一个转移的卡槽的节点，对原数组的操作是倒叙的。
        }
        int nextn = nextTab.length;
        
         /**
         * ForwardingNode是继承了Node的 但是他所有节点的hash值都是设置成了MOVED，标识着当前节点在扩容当中
         * above for explanation.
         */
        ForwardingNode<K,V> fwd = new ForwardingNode<K,V>(nextTab);
        boolean advance = true;
        boolean finishing = false; // to ensure sweep before committing nextTab
        for (int i = 0, bound = 0;;) {
            Node<K,V> f; int fh;
            while (advance) {
                <!--nextIndex 是开始拷贝的槽点位置 是从尾部开始 每次-1的 -->
                int nextIndex, nextBound;
                <!-- --i >= bound这个说明此次的循环结束 bound是每次执行到最后的那个值  finishing说明这个整个拷贝结束-->
                if (--i >= bound || finishing)
                    advance = false;
               <!--说明整个数组都拷贝结束了 index 已经是 到最开始的处  倒叙拷贝 的所以是判断小于等0 是算结束-->
                else if ((nextIndex = transferIndex) <= 0) {
                    i = -1;
                    advance = false;
                }
                else if (U.compareAndSwapInt
                         (this, TRANSFERINDEX, nextIndex,
                          nextBound = (nextIndex > stride ?
                                       nextIndex - stride : 0))) {
                    bound = nextBound;
                    i = nextIndex - 1;// 这边开始 每次减一
                    advance = false;
                }
            }
            if (i < 0 || i >= n || i + n >= nextn) {
                int sc;
                if (finishing) {
                    nextTable = null;//临时表 赋值为null
                    table = nextTab;//结束后 新的table 赋值
                    sizeCtl = (n << 1) - (n >>> 1);//
                    return;
                }
                if (U.compareAndSwapInt(this, SIZECTL, sc = sizeCtl, sc - 1)) {
                    if ((sc - 2) != resizeStamp(n) << RESIZE_STAMP_SHIFT)
                        return;
                    finishing = advance = true;
                    i = n; // recheck before commit
                }
            }
            else if ((f = tabAt(tab, i)) == null)
                advance = casTabAt(tab, i, null, fwd);
            else if ((fh = f.hash) == MOVED)
                advance = true; // already processed
            else {
                synchronized (f) {
                    if (tabAt(tab, i) == f) {
                        Node<K,V> ln, hn;
                        if (fh >= 0) {
                           //是链表的时候处理
                        }
                        else if (f instanceof TreeBin) {
                           //红黑树的时候处理
                        }
                    }
                }
            }
        }
    }
```
1. 首先从原数组的队尾开始进行拷贝；
2. 拷贝数组时，会把原数组的槽点锁住，使用的是`Synchronized`，这样原数组里面的数据就没法被修改，保证了线程安全，成功拷贝到新的数组后，把原数组的槽点设置为转移节点`move`;
3. 如果此时有数据`put`，当前槽点状态是转移节点也就是`move`，就会一直等待；
4. 直到原数组所有的节点被复制到新数组里，然后再把新数组赋值给数组容器，完成拷贝。
5. 总结：在数组扩容时，主要是利用`Synchronized`去锁住槽点，不让别的线程去操作，槽点复制成功后，会标识为转移节点，这样新的`put`的操作过来，看到槽点的状态是`move`，就会一直等待扩容完成后才会进行`put`操作。

## 5.get操作
```Java
public V get(Object key) {
        Node<K,V>[] tab; Node<K,V> e, p; int n, eh; K ek;
        int h = spread(key.hashCode());//获取hash值
        if ((tab = table) != null && (n = tab.length) > 0 &&
            (e = tabAt(tab, (n - 1) & h)) != null) {
            <!--根据hash值 找到槽点  然后看槽点的第一个节点 是否是要找到值 如果是直接返回-->
            if ((eh = e.hash) == h) {
                if ((ek = e.key) == key || (ek != null && key.equals(ek)))
                    return e.val;
            }
            <!--节点的hash值小于0 说明是转移节点 或者是红黑树-->
            else if (eh < 0)
                return (p = e.find(h, key)) != null ? p.val : null;
            <!--判断如果是链表 就遍历 查找-->
            while ((e = e.next) != null) {
                if (e.hash == h &&
                    ((ek = e.key) == key || (ek != null && key.equals(ek))))
                    return e.val;
            }
        }
        return null;
    }
```
**初看和HashMap的get方法并无差异，但是继续看以下的代码。**
```Java
**
     * The array of bins. Lazily initialized upon first insertion.
     * Size is always a power of two. Accessed directly by iterators.
     */
    transient volatile Node<K,V>[] table;

    /**
     * The next table to use; non-null only while resizing.
     */
    private transient volatile Node<K,V>[] nextTable;

    /**
     * Base counter value, used mainly when there is no contention,
     * but also as a fallback during table initialization
     * races. Updated via CAS.
     */
    private transient volatile long baseCount;

    /**
     * Table initialization and resizing control.  When negative, the
     * table is being initialized or resized: -1 for initialization,
     * else -(1 + the number of active resizing threads).  Otherwise,
     * when table is null, holds the initial table size to use upon
     * creation, or 0 for default. After initialization, holds the
     * next element count value upon which to resize the table.
     */
    private transient volatile int sizeCtl;

    /**
     * The next table index (plus one) to split while resizing.
     */
    private transient volatile int transferIndex;

    /**
     * Spinlock (locked via CAS) used when resizing and/or creating CounterCells.
     */
    private transient volatile int cellsBusy;

    /**
     * Table of counter cells. When non-null, size is a power of 2.
     */
    private transient volatile CounterCell[] counterCells;
```
**但其实类的成员变量都用了关键字volatile做了修饰，保证多线程直接的内存可见性。如果是基础类型，可以保证可见性，但如果是引用类型，对象的可见性是保证了，但是对象里的属性值如果修改了是没法知道的。**
**继续看数组容器里的Node节点代码。**
```Java
   static class Node<K,V> implements Map.Entry<K,V> {
        final int hash;
        final K key;
        volatile V val;
        volatile Node<K,V> next;

        Node(int hash, K key, V val, Node<K,V> next) {
            this.hash = hash;
            this.key = key;
            this.val = val;
            this.next = next;
        }
    }
```
**这里的Node节点里的val节点和下一个节点next都是使用了volatile做修饰的。也就是说在修改节点值时和插入节点值是，线程之间都是可见的。这样就可以在不加锁的情况下使用get方法能获取到最新的值，保证了多线程下的同步，保证了线程安全。**

[ConcurrentHashMap 怎么样去保证线程安全的, 读操作为什么不需要加锁？ - SegmentFault 思否](https://segmentfault.com/a/1190000023313702)
