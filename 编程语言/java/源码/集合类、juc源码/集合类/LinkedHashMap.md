# 概览

LinkedHashMap是按照`插入顺序`排序的Map结构。具有两大特性：

- 按照插⼊顺序进⾏访问
- 实现了访问最少最先删除功能，其⽬的是把很久都没有访问的 key ⾃动删除。

<img src="https://s2.ax1x.com/2020/02/01/1JpBQS.png" style="zoom:67%;" />

## 属性

```java
// 链表头
transient LinkedHashMap.Entry<K,V> head;

// 链表尾
transient LinkedHashMap.Entry<K,V> tail;

// 继承 Node，为数组的每个元素增加了 before 和 after 属性
static class Entry<K,V> extends HashMap.Node<K,V> {
    Entry<K,V> before, after;
    Entry(int hash, K key, V value, Node<K,V> next) {
        super(hash, key, value, next);
    }
}

// 控制两种访问模式的字段，默认 false
// true 按照访问顺序，会把经常访问的 key 放到队尾
// false 按照插入顺序提供访问
final boolean accessOrder;
```

LinkedHashMap相比于HashMap区别在于`Entry`中多了`before、after`。



# 源码

## 新增

重写了`newNode`方法，也就是创建新节点。

```java
// 新增节点，并追加到链表的尾部
Node<K,V> newNode(int hash, K key, V value, Node<K,V> e) {
    // 新增节点
    LinkedHashMap.Entry<K,V> p =
        new LinkedHashMap.Entry<K,V>(hash, key, value, e);
    // 追加到链表的尾部
    linkNodeLast(p);
    return p;
}
// link at the end of list
private void linkNodeLast(LinkedHashMap.Entry<K,V> p) {
    LinkedHashMap.Entry<K,V> last = tail;
    // 新增节点等于位节点
    tail = p;
    // last 为空，说明链表为空，首尾节点相等
    if (last == null)
        head = p;
    // 链表有数据，直接建立新增节点和上个尾节点之间的前后关系即可
    else {
        p.before = last;
        last.after = p;
    }
}
```

  

# 拓展

## LRU(最少访问)

通过LinkedHashMap我们可以很方便的实现LRU。

构造方法有一个参数`accessOrder`，这个参数默认为false，从它的注释说明来看，当参数为true时，按照访问顺序(也就是访问多的移到链表末尾)，当参数为false时，按照插入顺序

```java
    public LinkedHashMap(int initialCapacity,   // 初始容量
                         float loadFactor,      // 负载因子
                         boolean accessOrder) {  // 访问策略
        super(initialCapacity, loadFactor);
        this.accessOrder = accessOrder;
    }

```

设置为`true`后，get方法会把查询到的元素移到对列末尾。

```java
public V get(Object key) {
    Node<K,V> e;
    // 调用 HashMap  get 方法
    if ((e = getNode(hash(key), key)) == null)
        return null;
    // 如果设置了 LRU 策略
    if (accessOrder)
    // 这个方法把当前 key 移动到队尾
        afterNodeAccess(e);
    return e.value;
}
```

当然不仅是`get`方法，还有其他方法，执⾏ getOrDefault、compute、computeIfAbsent、computeIfPresent、merge ⽅法时也会移到末尾。`也就是把经常访问的元素不断的移到末尾，这个头部就是最少访问的元素了`。



当然`LinkedHashMap`还提供了一个方法，也就是移除最少访问元素。

```java
// 删除很少被访问的元素，被 HashMap 的 put 方法所调用
void afterNodeInsertion(boolean evict) { 
    // 得到元素头节点
    LinkedHashMap.Entry<K,V> first;
    // removeEldestEntry 来控制删除策略，如果队列不为空，并且删除策略允许删除的情况下，删除头节点
    if (evict && (first = head) != null && removeEldestEntry(first)) {
        K key = first.key;
        // removeNode 删除头节点
        removeNode(hash(key), key, null, false, true);
    }
}

protected boolean removeEldestEntry(Map.Entry<K,V> eldest) {
        return false;
}
```

## 案例

```java
public class TestLinkedHashMap02 {
        public static void main(String[] args) {
            LruCache cache=new LruCache(3);
            cache.put("A", 100);
            cache.put("B", 200);
            cache.put("C", 300);
            System.out.println(cache);
            cache.get("A");
            cache.put("D", 400);
            System.out.println(cache);//CAD
        }
}
/**
 * 基于LinkedHashMap实现一个基于LRU算法的缓存设计
 * 思考:
 * 1)MyBatis中缓存的设计
 * 2)Spring中的缓存设计
 * 3)EhCache中缓存设计
 * 4)redis中缓存设计
 * 5).....................
 * 自定义LruCache实现
 * 1)有容量限制
 * 2)容器满了要基于LRU算法移出元素
 */
class LruCache extends LinkedHashMap<String,Object>{
    private static final long serialVersionUID = -1838771409260368496L;
    /**设计缓存的最大容量*/
    private int maxCapacity;
    public LruCache(int maxCapacity) {
        super(maxCapacity,0.75f, true);
        this.maxCapacity=maxCapacity;
    }
    /**
     * 方法的返回值决定了容器在满的时是否移除元素.
     * true:表示要移除
     * false:表示不移除
     */
    @Override
    protected boolean removeEldestEntry(java.util.Map.Entry<String, Object> eldest) {
        return size()>maxCapacity;
    }
}
```

