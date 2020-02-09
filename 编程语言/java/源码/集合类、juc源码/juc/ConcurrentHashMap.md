# 概述

提起线程安全的Map，我们第一个想起来的就是`ConcurrentHashMap`。`ConcurrentHashMap`内部封装了各种锁和数据结构来确保线程安全。

**我们从类注释信息上可以得到以下信息**

1. 所有的操作都是线程安全的，我们在使⽤时，⽆需再加锁；
2. 多个线程同时进⾏ put、remove 等操作时并不会阻塞，可以同时进⾏，和 HashTable 不同，
HashTable 在操作时，会锁住整个 Map；
3. 迭代过程中，即使 Map 结构被修改，也不会抛 ConcurrentModificationException 异常；
4. 除了数组 + 链表 + 红⿊树的基本结构外，新增了转移节点，是为了保证扩容时的线程安全的
节点；
5. 提供了很多 Stream 流式⽅法，⽐如说：forEach、search、reduce 等等。

`ConcurrentHashMap`继承与`HashMap`，底层数据结构唯一不同的就是`ConcurrentHashMap`新增了`转移节点`。

# 源码

## put

1. 如果数组为空，初始化，初始化完成之后，⾛ 2；
2. 计算当前槽点有没有值，没有值的话，`cas 创建`，失败继续⾃旋（for 死循环），直到成功，
槽点有值的话，⾛ 3；
3. 如果槽点是`转移节点(正在扩容)`，就会⼀直⾃旋等待扩容完成之后再新增，不是转移节点⾛
4；
4. 槽点有值的，`先锁定当前槽点，保证其余线程不能操作`，如果是链表，新增值到链表的尾部，
如果是红⿊树，使⽤红⿊树新增的⽅法新增；
5. 新增完成之后 check 需不需要扩容，需要的话去扩容。

```java
final V putVal(K key, V value, boolean onlyIfAbsent) {
    if (key == null || value == null) throw new NullPointerException();
    //计算hash
    int hash = spread(key.hashCode());
    int binCount = 0;
    for (Node<K,V>[] tab = table;;) { // cas?
        Node<K,V> f; int n, i, fh;
        //table是空的，进行初始化
        if (tab == null || (n = tab.length) == 0)
            tab = initTable();
        //如果当前索引位置没有值，直接创建
        else if ((f = tabAt(tab, i = (n - 1) & hash)) == null) {
            //cas 在 i 位置创建新的元素，当 i 位置是空时，即能创建成功，结束for自循，否则继续自旋
            if (casTabAt(tab, i, null,
                         new Node<K,V>(hash, key, value, null)))
                break;                   // no lock when adding to empty bin
        }
        //如果当前槽点是转移节点，表示该槽点正在扩容，就会一直等待扩容完成
        //转移节点的 hash 值是固定的，都是 MOVED
        else if ((fh = f.hash) == MOVED)
           //帮其扩容
            tab = helpTransfer(tab, f);
        //槽点上有值的
        else {
            V oldVal = null;
            //锁定当前槽点，其余线程不能操作，保证了安全
            synchronized (f) {
                //这里再次判断 i 索引位置的数据没有被修改
                //binCount 被赋值的话，说明走到了修改表的过程里面
                if (tabAt(tab, i) == f) {
                    //链表
                    if (fh >= 0) {
                        binCount = 1;
                        for (Node<K,V> e = f;; ++binCount) {
                            K ek;
                            //值有的话，直接返回
                            if (e.hash == hash &&
                                ((ek = e.key) == key ||
                                 (ek != null && key.equals(ek)))) {
                                oldVal = e.val;
                                if (!onlyIfAbsent)
                                    e.val = value;
                                break;
                            }
                            Node<K,V> pred = e;
                            //把新增的元素赋值到链表的最后，退出自旋
                            if ((e = e.next) == null) {
                                pred.next = new Node<K,V>(hash, key,
                                                          value, null);
                                break;
                            }
                        }
                    }
                    //红黑树，这里没有使用 TreeNode,使用的是 TreeBin，TreeNode 只是红黑树的一个节点
                    //TreeBin 持有红黑树的引用，并且会对其加锁，保证其操作的线程安全
                    else if (f instanceof TreeBin) {
                        Node<K,V> p;
                        binCount = 2;
                        //满足if的话，把老的值给oldVal
                        //在putTreeVal方法里面，在给红黑树重新着色旋转的时候
                        //会锁住红黑树的根节点
                        if ((p = ((TreeBin<K,V>)f).putTreeVal(hash, key,
                                                       value)) != null) {
                            oldVal = p.val;
                            if (!onlyIfAbsent)
                                p.val = value;
                        }
                    }
                }
            }
            //binCount不为空，并且 oldVal 有值的情况，说明已经新增成功了
            if (binCount != 0) {
                // 链表是否需要转化成红黑树
                if (binCount >= TREEIFY_THRESHOLD)
                    treeifyBin(tab, i);
                if (oldVal != null)
                    return oldVal;
                //这一步几乎走不到。槽点已经上锁，只有在红黑树或者链表新增失败的时候
                //才会走到这里，这两者新增都是自旋的，几乎不会失败
                break;
            }
        }
    }
    //check 容器是否需要扩容，如果需要去扩容，调用 transfer 方法去扩容
    //如果已经在扩容中了，check有无完成
    addCount(1L, binCount);
    return null;
}

// hash计算
static final int spread(int h) {
   return (h ^ (h >>> 16)) & HASH_BITS;
}
```

**数组初始化**

数组初始化时，⾸先通过⾃旋来保证⼀定可以初始化成功，然后通过 CAS 设置 `SIZECTL`变量的值，来保证同⼀时刻只能有⼀个线程对数组进⾏初始化，CAS 成功之后，还会再次判断当前数组是否已经初始化完成，如果已经初始化完成，就不会再次初始化，通过⾃旋 + CAS + 双重 check 等⼿段保证了数组初始化时的线程安全

```java
//初始化 table，通过对 sizeCtl 的变量赋值来保证数组只能被初始化一次
private final Node<K,V>[] initTable() {
    Node<K,V>[] tab; int sc;
    //通过自旋保证初始化成功
    while ((tab = table) == null || tab.length == 0) {
        // 小于 0 代表有线程正在初始化，释放当前 CPU 的调度权，重新发起锁的竞争
        if ((sc = sizeCtl) < 0)
            Thread.yield(); // lost initialization race; just spin
        // CAS 赋值保证当前只有一个线程在初始化，-1 代表当前只有一个线程能初始化
        // 保证了数组的初始化的安全性
        else if (U.compareAndSwapInt(this, SIZECTL, sc, -1)) {
            try {
                // 很有可能执行到这里的时候，table 已经不为空了，这里是双重 check
                if ((tab = table) == null || tab.length == 0) {
                    // 进行初始化
                    int n = (sc > 0) ? sc : DEFAULT_CAPACITY;
                    @SuppressWarnings("unchecked")
                    Node<K,V>[] nt = (Node<K,V>[])new Node<?,?>[n];
                    table = tab = nt;
                  // 相当于 n*0.75, 相当于扩容阈值。
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

**新增节点的线程安全**

1. 通过自旋死循环来保证新增一定成功，一旦新增成功就可以退出当前死循环。

   ```java
   for (Node<K,V>[] tab = table;;) 
   ```

2. 当前槽点为空是，就采用cas算法进行直接赋值。`如果恰好槽点已经被其他线程赋值，当前 CAS 操作失败，会再次执⾏ for ⾃旋，再⾛槽点有值的 put 流程，这⾥就是⾃旋 + CAS 的结合`

   ```java
   //原子的执行如下逻辑：如果tab[i]==c,则设置tab[i]=v，并返回ture.否则返回false
   if (casTabAt(tab, i, null,
                                new Node<K,V>(hash, key, value, null)))
                       break;                   // no lock when adding to empt
   ```

   [ConcurrentHashMap源码分析](https://www.jianshu.com/p/5bc70d9e5410)

3. 当前槽点有值，锁住当前槽点

   ```java
     synchronized (f) {}
   ```

通过以上 3 点，保证了在各种情况下的新增（不考虑扩容的情况下），都是线程安全的，通过自旋 + CAS + 锁三大姿势。

### 扩容

ConcurrentHashMap 的扩容时机和 HashMap 相同，都是在 put 方法的最后一步检查是否需要扩容，如果需要则进行扩容，但两者扩容的过程完全不同

1. 首先需要把老数组的值全部拷贝到扩容之后的新数组上，先从数组的队尾开始拷贝；
2. 拷贝数组的槽点时，先把原数组槽点锁住，保证原数组槽点不能操作，成功拷贝到新数组时，把原数组槽点赋值为转移节点；
3. 这时如果有新数据正好需要 put 到此槽点时，发现槽点为转移节点，就会一直等待，所以在扩容完成之前，该槽点对应的数据是不会发生变化的；
4. 从数组的尾部拷贝到头部，每拷贝成功一次，就把原数组中的节点设置成转移节点；
5. 直到所有数组数据都拷贝到新数组时，直接把新数组整个赋值给数组容器，拷贝完成。

```java
// 扩容主要分 2 步，第一新建新的空数组，第二移动拷贝每个元素到新数组中去
// tab：原数组，nextTab：新数组
private final void transfer(Node<K,V>[] tab, Node<K,V>[] nextTab) {
    // 老数组的长度
    int n = tab.length, stride;
    if ((stride = (NCPU > 1) ? (n >>> 3) / NCPU : n) < MIN_TRANSFER_STRIDE)
        stride = MIN_TRANSFER_STRIDE; // subdivide range
    // 如果新数组为空，初始化，大小为原数组的两倍，n << 1
    if (nextTab == null) {            // initiating
        try {
            @SuppressWarnings("unchecked")
            Node<K,V>[] nt = (Node<K,V>[])new Node<?,?>[n << 1];
            nextTab = nt;
        } catch (Throwable ex) {      // try to cope with OOME
            sizeCtl = Integer.MAX_VALUE;
            return;
        }
        nextTable = nextTab;
        transferIndex = n;
    }
    // 新数组的长度
    int nextn = nextTab.length;
    // 代表转移节点，如果原数组上是转移节点，说明该节点正在被扩容
    ForwardingNode<K,V> fwd = new ForwardingNode<K,V>(nextTab);
    boolean advance = true;
    boolean finishing = false; // to ensure sweep before committing nextTab
    // 无限自旋，i 的值会从原数组的最大值开始，慢慢递减到 0
    for (int i = 0, bound = 0;;) {
        Node<K,V> f; int fh;
        while (advance) {
            int nextIndex, nextBound;
            // 结束循环的标志
            if (--i >= bound || finishing)
                advance = false;
            // 已经拷贝完成
            else if ((nextIndex = transferIndex) <= 0) {
                i = -1;
                advance = false;
            }
            // 每次减少 i 的值
            else if (U.compareAndSwapInt
                     (this, TRANSFERINDEX, nextIndex,
                      nextBound = (nextIndex > stride ?
                                   nextIndex - stride : 0))) {
                bound = nextBound;
                i = nextIndex - 1;
                advance = false;
            }
        }
        // if 任意条件满足说明拷贝结束了
        if (i < 0 || i >= n || i + n >= nextn) {
            int sc;
            // 拷贝结束，直接赋值，因为每次拷贝完一个节点，都在原数组上放转移节点，所以拷贝完成的节点的数据一定不会再发生变化。
            // 原数组发现是转移节点，是不会操作的，会一直等待转移节点消失之后在进行操作。
            // 也就是说数组节点一旦被标记为转移节点，是不会再发生任何变动的，所以不会有任何线程安全的问题
            // 所以此处直接赋值，没有任何问题。
            if (finishing) {
                nextTable = null;
                table = nextTab;
                sizeCtl = (n << 1) - (n >>> 1);
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
                // 进行节点的拷贝
                if (tabAt(tab, i) == f) {
                    Node<K,V> ln, hn;
                    if (fh >= 0) {
                        int runBit = fh & n;
                        Node<K,V> lastRun = f;
                        for (Node<K,V> p = f.next; p != null; p = p.next) {
                            int b = p.hash & n;
                            if (b != runBit) {
                                runBit = b;
                                lastRun = p;
                            }
                        }
                        if (runBit == 0) {
                            ln = lastRun;
                            hn = null;
                        }
                        else {
                            hn = lastRun;
                            ln = null;
                        }
                        // 如果节点只有单个数据，直接拷贝，如果是链表，循环多次组成链表拷贝
                        for (Node<K,V> p = f; p != lastRun; p = p.next) {
                            int ph = p.hash; K pk = p.key; V pv = p.val;
                            if ((ph & n) == 0)
                                ln = new Node<K,V>(ph, pk, pv, ln);
                            else
                                hn = new Node<K,V>(ph, pk, pv, hn);
                        }
                        // 在新数组位置上放置拷贝的值
                        setTabAt(nextTab, i, ln);
                        setTabAt(nextTab, i + n, hn);
                        // 在老数组位置上放上 ForwardingNode 节点
                        // put 时，发现是 ForwardingNode 节点，就不会再动这个节点的数据了
                        setTabAt(tab, i, fwd);
                        advance = true;
                    }
                    // 红黑树的拷贝
                    else if (f instanceof TreeBin) {
                        // 红黑树的拷贝工作，同 HashMap 的内容，代码忽略
                        …………
                        // 在老数组位置上放上 ForwardingNode 节点
                        setTabAt(tab, i, fwd);
                        advance = true;
                    }
                }
            }
        }
    }
}
```

保证线程安全主要是以下几点

1. 拷贝槽点时，会把原数组的槽点锁住；
2. 拷贝成功之后，会把原数组的槽点设置成转移节点，这样如果有数据需要 put 到该节点时，发现该槽点是转移节点，会一直等待，直到扩容成功之后，才能继续 put，可以参考 put 方法中的 helpTransfer 方法；
3. 从尾到头进行拷贝，拷贝成功就把原数组的槽点设置成转移节点。
4. 等扩容拷贝都完成之后，直接把新数组的值赋值给数组容器，之前等待 put 的数据才能继续 put。

