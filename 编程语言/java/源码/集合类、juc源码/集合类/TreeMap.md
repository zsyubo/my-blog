# 概览

TreeMap 是一个**有序(默认是自然顺序，比如ASCII顺序)的key-value集合**，它是通过红黑树实现的。默认排序是调用存储元素的`compareTo`来比较。当然也可以在传入`Comparator`对象。

```java
        TreeMap<String,String> te= new TreeMap<>(new Comparator<String>(){
            /*
             * int compare(Object o1, Object o2) 返回一个基本类型的整型，
             * 返回负数表示：o1 小于o2，
             * 返回0 表示：o1和o2相等，
             * 返回正数表示：o1大于o2。
             */
            public int compare(String o1, String o2) {
                //指定排序器按照降序排列
                return o2.compareTo(o1);
            }
        });
```

同时，TreeMap是线程不安全的map

## 属性

```java
//比较器，如果外部有传进来 Comparator 比较器，首先用外部的
//如果外部比较器为空，则使用 key 自己实现的 Comparable#compareTo 方法
//比较手段和上面日常工作中的比较 demo 是一致的
private final Comparator<? super K> comparator;

//红黑树的根节点
private transient Entry<K,V> root;

//红黑树的已有元素大小
private transient int size = 0;

//树结构变化的版本号，用于迭代过程中的快速失败场景
private transient int modCount = 0;

//红黑树的节点
static final class Entry<K,V> implements Map.Entry<K,V> {}
```



#源码

## 新增

```java
 public V put(K key, V value) {
   			// 1. 判断红⿊树的节点是否为空，为空的话，新增的节点直接作为根节点，代码如下
         Entry<K,V> t = root;
        //红黑树根节点为空，直接新建
        if (t == null) {
            // compare 方法限制了 key 不能为 null
            compare(key, key); // type (and possibly null) check
            // 成为根节点
            root = new Entry<>(key, value, null);
            size = 1;
            modCount++;
            return null;
        }
        int cmp;
        Entry<K,V> parent;
        // 2. 根据红⿊树左⼩右⼤的特性，进⾏判断，找到应该新增节点的⽗节点
       	Comparator<? super K> cpr = comparator;
        if (cpr != null) {
            //自旋找到 key 应该新增的位置，就是应该挂载那个节点的头上
            do {
                //一次循环结束时，parent 就是上次比过的对象
                parent = t;
                // 通过 compare 来比较 key 的大小
                cmp = cpr.compare(key, t.key);
                //key 小于 t，把 t 左边的值赋予 t，因为红黑树左边的值比较小，循环再比
                if (cmp < 0)
                    t = t.left;
                //key 大于 t，把 t 右边的值赋予 t，因为红黑树右边的值比较大，循环再比
                else if (cmp > 0)
                    t = t.right;
                //如果相等的话，直接覆盖原值
                else
                    return t.setValue(value);
                // t 为空，说明已经到叶子节点了
            } while (t != null);
        }else {
            if (key == null)
                throw new NullPointerException();
            @SuppressWarnings("unchecked")
                Comparable<? super K> k = (Comparable<? super K>) key;
            do {
                parent = t;
                cmp = k.compareTo(t.key);
                if (cmp < 0)
                    t = t.left;
                else if (cmp > 0)
                    t = t.right;
                else
                    return t.setValue(value);
            } while (t != null);
        }
        Entry<K,V> e = new Entry<>(key, value, parent);
   // 3. 在⽗节点的左边或右边插⼊新增节点
        //cmp 代表最后一次对比的大小，小于 0 ，代表 e 在上一节点的左边
        if (cmp < 0)
            parent.left = e;
        //cmp 代表最后一次对比的大小，大于 0 ，代表 e 在上一节点的右边，相等的情况第二步已经处理了。
        else
            parent.right = e;
        fixAfterInsertion(e);
        size++;
        modCount++;
        return null;
    }
```

## 查找

按照红黑树特性进行查找