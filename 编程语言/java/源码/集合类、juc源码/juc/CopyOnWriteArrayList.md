# 概览

ArrayList是线程不安全的，所以JUC提供了线程安全的List：`CopyOnWriteArrayList`，它具有以下特性：

1. 线程安全的，多线程环境下可以直接使⽤，⽆需加锁；

2. 通过锁 + 数组拷⻉ + volatile 关键字保证了线程安全；

3. 每次数组操作，都会把数组拷⻉⼀份出来，在新数组上进⾏操作，操作成功之后再赋值回去。

   

`CopyOnWriteArrayList`与`ArrayList`底层相似，底层都是数组。但是`CopyOnWriteArrayList`操作数组为了保证线程安全，增加了下列操作：

1. 加锁；
2. 从原数组中拷⻉出新数组；
3. 在新数组上进⾏操作，并把新数组赋值给数组容器；
4. 解锁。

除了加锁外，底层存储元素的数组使用了`volatile`修饰。

```java
private transient volatile Object[] array;
```

`整体上来说，CopyOnWriteArrayList 就是利⽤锁 + 数组拷⻉ + volatile 关键字保证了 List 的线程安全`

# 源码

## 新增

```java
// 添加元素到数组尾部
public boolean add(E e) {
    final ReentrantLock lock = this.lock;
    // 加锁
    lock.lock();
    try {
        // 得到所有的原数组
        Object[] elements = getArray();
        int len = elements.length;
        // 拷贝到新数组里面，新数组的长度是 + 1 的，因为新增会多一个元素
        Object[] newElements = Arrays.copyOf(elements, len + 1);
        // 在新数组中进行赋值，新元素直接放在数组的尾部
        newElements[len] = e;
        // 替换掉原来的数组
        setArray(newElements);
        return true;
    // finally 里面释放锁，保证即使 try 发生了异常，仍然能够释放锁   
    } finally {
        lock.unlock();
    }
}
```

源码比较简单，核心拷贝一份，操作完成后再替换原数组。

**新增到指定位置**

```java
    public E set(int index, E element) {
        final ReentrantLock lock = this.lock;
        lock.lock();
        try {
            Object[] elements = getArray();
          	// 获取原值
            E oldValue = get(elements, index);

            if (oldValue != element) {
              	// 拷贝一份和原数组一样大小的数组。
                int len = elements.length;
                Object[] newElements = Arrays.copyOf(elements, len);
                // set值
                newElements[index] = element;
                // 替换掉原数组
                setArray(newElements);
            } else {
                // Not quite a no-op; ensures volatile write semantics
                setArray(elements);
            }
            return oldValue;
        } finally {
            lock.unlock();
        }
    }
```

也没撒好说的。

## 删除

```java
// 删除某个索引位置的数据
public E remove(int index) {
    final ReentrantLock lock = this.lock;
    // 加锁
    lock.lock();
    try {
        Object[] elements = getArray();
        int len = elements.length;
        // 先得到老值
        E oldValue = get(elements, index);
        int numMoved = len - index - 1;
        // 如果要删除的数据正好是数组的尾部，直接删除
        if (numMoved == 0)
            setArray(Arrays.copyOf(elements, len - 1));
        else {
            // 如果删除的数据在数组的中间，分三步走
            // 1. 设置新数组的长度减一，因为是减少一个元素
            // 2. 从 0 拷贝到数组新位置
            // 3. 从新位置拷贝到数组尾部
            Object[] newElements = new Object[len - 1];
            System.arraycopy(elements, 0, newElements, 0, index);
            System.arraycopy(elements, index + 1, newElements, index,
                             numMoved);
            setArray(newElements);
        }
        return oldValue;
    } finally {
        lock.unlock();
    }
}
```

