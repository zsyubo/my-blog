# 概览

HashMap的底层数据结构式：数组+链表+红黑树。`当链表长度大于8并且数组大小大于64时将转化为红黑树`。当红黑树大小小于6时，红黑树将转化为链表。整体结构如下。

<img src="https://img1.sycdn.imooc.com/5d5fc7cc0001ec3211040928.png" style="zoom:63%;" />

**重点**

- 允许key为null，同时线程不安全。
- 负载因子默认是0.75，是大牛为了均衡而考虑的，如果负载因子过大，会减少空间开销，但是会增大查找成本（增大hash冲突）
- 建议初始化时指定数组长度，避免在使用过程中扩容。
- 在迭代过程中，如果HashMap的结构被修改，会快速失败。

**属性**

```java
 //初始容量为 16
 static final int DEFAULT_INITIAL_CAPACITY = 1 << 4;

 //最大容量
 static final int MAXIMUM_CAPACITY = 1 << 30;

 //负载因子默认值
 static final float DEFAULT_LOAD_FACTOR = 0.75f;
 
 //桶上的链表长度大于等于8时，链表转化成红黑树
 static final int TREEIFY_THRESHOLD = 8;

 //桶上的红黑树大小小于等于6时，红黑树转化成链表
 static final int UNTREEIFY_THRESHOLD = 6;

 //当数组容量大于 64 时，链表才会转化成红黑树
 static final int MIN_TREEIFY_CAPACITY = 64;

 //记录迭代过程中 HashMap 结构是否发生变化，如果有变化，迭代时会 fail-fast
 transient int modCount;

 //HashMap 的实际大小，可能不准(因为当你拿到这个值的时候，可能又发生了变化)
 transient int size;

 //存放数据的数组
 transient Node<K,V>[] table;

 // 扩容的门槛，有两种情况
 // 如果初始化时，给定数组大小的话，通过 tableSizeFor 方法计算，数组大小永远接近于 2 的幂次方，比如你给定初始化大小 19，实际上初始化大小为 32，为 2 的 5 次方。
 // 如果是通过 resize 方法进行扩容，大小 = 数组容量 * 0.75
 int threshold;

 //链表的节点
 static class Node<K,V> implements Map.Entry<K,V> {
 
 //红黑树的节点
 static final class TreeNode<K,V> extends LinkedHashMap.Entry<K,V> {
```

# 源码

## 新增

1. 空数组有⽆初始化，没有的话初始化；
2. 如果通过 key 的 hash 能够直接找到值，跳转到 6，否则到 3；
3. 如果 hash 冲突，两种解决⽅案：链表 or 红⿊树；
4. 如果是链表，递归循环，把新元素追加到队尾；
5. 如果是红⿊树，调⽤红⿊树新增的⽅法；
6. 通过 2、4、5 将新元素追加成功，再根据 onlyIfAbsent 判断是否需要覆盖；
7. 判断是否需要扩容，需要扩容进⾏扩容，结束。

**图解**

<img src="https://img1.sycdn.imooc.com/5d5fc7e200016af809121188.jpg" style="zoom:50%;" />



```java
    public V put(K key, V value) {
        // key 做一次hash
        return putVal(hash(key), key, value, false, true);
    }
```

hash源码

```java
    static final int hash(Object key) {
        int h;
        return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
    }
```

`putVal`源码

```java
// 入参 hash：通过 hash 算法计算出来的值。
// 入参 onlyIfAbsent：false 表示即使 key 已经存在了，仍然会用新值覆盖原来的值，默认为 false
final V putVal(int hash, K key, V value, boolean onlyIfAbsent,
               boolean evict) {
    // n 表示数组的长度，i 为数组索引下标，p 为 i 下标位置的 Node 值
    Node<K,V>[] tab; Node<K,V> p; int n, i;
    //如果数组为空，使用 resize 方法初始化
    if ((tab = table) == null || (n = tab.length) == 0)
        n = (tab = resize()).length;
    // 如果当前索引位置是空的，直接生成新的节点在当前索引位置上
    if ((p = tab[i = (n - 1) & hash]) == null)
        tab[i] = newNode(hash, key, value, null);
    // 如果当前索引位置有值的处理方法，即我们常说的如何解决 hash 冲突
    else {
        // e 当前节点的临时变量
        Node<K,V> e; K k;
        // 如果 key 的 hash 和值都相等，直接把当前下标位置的 Node 值赋值给临时变量
        if (p.hash == hash &&
            ((k = p.key) == key || (key != null && key.equals(k))))
            e = p;
        // 如果是红黑树，使用红黑树的方式新增
        else if (p instanceof TreeNode)
            e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);
        // 是个链表，把新节点放到链表的尾端
        else {
            // 自旋
            for (int binCount = 0; ; ++binCount) {
                // e = p.next 表示从头开始，遍历链表
                // p.next == null 表明 p 是链表的尾节点
                if ((e = p.next) == null) {
                    // 把新节点放到链表的尾部 
                    p.next = newNode(hash, key, value, null);
                    // 当链表的长度大于等于 8 时，链表转红黑树
                    if (binCount >= TREEIFY_THRESHOLD - 1)
                        treeifyBin(tab, hash);
                    break;
                }
                // 链表遍历过程中，发现有元素和新增的元素相等，结束循环
                if (e.hash == hash &&
                    ((k = e.key) == key || (key != null && key.equals(k))))
                    break;
                //更改循环的当前元素，使 p 在遍历过程中，一直往后移动。
                p = e;
            }
        }
        // 说明新节点的新增位置已经找到了
        if (e != null) {
            V oldValue = e.value;
            // 当 onlyIfAbsent 为 false 时，才会覆盖值 
            if (!onlyIfAbsent || oldValue == null)
                e.value = value;
            afterNodeAccess(e);
            // 返回老值
            return oldValue;
        }
    }
    // 记录 HashMap 的数据结构发生了变化
    ++modCount;
    //如果 HashMap 的实际大小大于扩容的门槛，开始扩容
    if (++size > threshold)
        resize();
    afterNodeInsertion(evict);
    return null;
}
```

## 查找

查找分为3步

1. 根据 hash 算法定位数组的索引位置，equals 判断当前节点是否是我们需要寻找的 key，是的
   话直接返回，不是的话往下。

2. 判断当前节点有⽆ next 节点，有的话判断是链表类型，还是红⿊树类型。

3. 分别⾛链表和红⿊树不同类型的查找⽅法。

## remove

