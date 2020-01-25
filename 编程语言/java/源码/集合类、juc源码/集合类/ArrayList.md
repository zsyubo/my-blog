# 整体结构

ArrayList是一个动态数组，底层存储元素是一个数组结构。允许`add(null)`。ArrayList有几个重要的知识点：

1. `DEFAULT_CAPACITY`:初始化容量
2. `size`表示当前数组的大小,类型 int,没有使用 volatile 修饰,非线程安全的;
3. `elementData`为存储元素的数组。
4. 初始化不会立即初始化数组，而是默认指向一个空数组。如果初始化指定数组大小，那么在实例化时会直接初始化数组。
5. 扩容为原数组长度的1.5倍。

数组的优点就是`add、get、size`等的操作为O(1);

ArrayList是非线程安全的，比如里面的变量如`elementData、size`都未进行任何同步操作。并发安全的动态数组有`Collections#synchronizedList;`

ArrayList不允许在for循环的进行删除元素的操作，正确地方式时使用迭代器, 然后通过迭代器来进行`remove`。

```java
Iterator<String> it = ints.iterator();   
while (it.hasNext()){                    
    String s  = it.next();               
    if (s.equals("3")){
      // 注意是使用迭代器的remove
        it.remove();                     
        continue;                        
    }                                    
    System.out.println(s);               
}                                        
```

# 源码解析

## 初始化

```java
private static final Object[] DEFAULTCAPACITY_EMPTY_ELEMENTDATA = {};

//无参数直接初始化，数组大小为空
public ArrayList() {
   // 赋予一个空数组
    this.elementData = DEFAULTCAPACITY_EMPTY_ELEMENTDATA;
}
// 传入数组大小
public ArrayList(int initialCapacity) {
     // 初始化长度大于0
     if (initialCapacity > 0) {
        // 实例化 elementData 长度为指定的长度
        this.elementData = new Object[initialCapacity];
     } else if (initialCapacity == 0) {  // 如果传如0 ，就让他等于长度为0 的数组
        this.elementData = EMPTY_ELEMENTDATA;
     } else {
        // 小于0 抛出异常
        throw new IllegalArgumentException("Illegal Capacity: "+
                                               initialCapacity);
     }
}

//指定初始数据初始化
public ArrayList(Collection<? extends E> c) {
    //elementData 是保存数组的容器，默认为 null
    elementData = c.toArray();
    //如果给定的集合（c）数据有值
    if ((size = elementData.length) != 0) {
        // c.toArray might (incorrectly) not return Object[] (see 6260652)
        //如果集合元素类型不是 Object 类型，我们会转成 Object
        if (elementData.getClass() != Object[].class) {
            elementData = Arrays.copyOf(elementData, size, Object[].class);
        }
    } else {
        // 给定集合（c）无值，则默认空数组
        this.elementData = EMPTY_ELEMENTDATA;
    }
}
```

从里面可以看出，默认初始化不会进行任何操作，`只有传入初始化大小是，才会去实例化数组。`

## 新增、扩容

新增操作主要为两步：

1. 判断是否需要进行扩容
2. 赋值。

```java
    public boolean add(E e) {
        // 判断容量是否能装下 新增数据，如果不足就扩容， 默认 为 先容量*1.5
        ensureCapacityInternal(size + 1);  // Increments modCount!!
        elementData[size++] = e;
        return true;
    }
```

`ensureCapacityInternal`里面的逻辑就是判断数组是否能够存放新增的元素

```java
 private void ensureCapacityInternal(int minCapacity) {
        ensureExplicitCapacity(calculateCapacity(elementData, minCapacity));
  }
```

`calculateCapacity(elementData, minCapacity)`主要是判断底层数组是空数组，然后返回数组所需容量大小。

```java
 private static int calculateCapacity(Object[] elementData, int minCapacity) {
        // 如果底层数组为空，这去判断 传入的 数组长度 与默认长度谁大
        if (elementData == DEFAULTCAPACITY_EMPTY_ELEMENTDATA) {
            return Math.max(DEFAULT_CAPACITY, minCapacity);
        }
        return minCapacity;
    }
```

具体扩容方法

```java
private void ensureExplicitCapacity(int minCapacity) {
  //记录数组被修改
  modCount++;
  // 如果我们期望的最小容量大于目前数组的长度，那么就扩容
  if (minCapacity - elementData.length > 0)
    grow(minCapacity);
}
//扩容，并把现有数据拷贝到新的数组里面去
private void grow(int minCapacity) {
  int oldCapacity = elementData.length;
  // oldCapacity >> 1 是把 oldCapacity 除以 2 的意思
  int newCapacity = oldCapacity + (oldCapacity >> 1);

  // 如果扩容后的值 < 我们的期望值，扩容后的值就等于我们的期望值
  if (newCapacity - minCapacity < 0)
    newCapacity = minCapacity;

  // 如果扩容后的值 > jvm 所能分配的数组的最大值，那么就用 Integer 的最大值
  if (newCapacity - MAX_ARRAY_SIZE > 0)
    newCapacity = hugeCapacity(minCapacity);
 
  // 通过复制进行扩容
  elementData = Arrays.copyOf(elementData, newCapacity);
}

private static final int MAX_ARRAY_SIZE = Integer.MAX_VALUE - 8;
// 判断数组大小是否超出最大值。
private static int hugeCapacity(int minCapacity) {
  // 如果传入小于0， 抛出
  if (minCapacity < 0) // overflow
    throw new OutOfMemoryError();
  // 防止int类型溢出。如果传入长度大于 最大长度，则返回  int maxm,  小于则返回MAX_ARRAY_SIZE
  return (minCapacity > MAX_ARRAY_SIZE) ?
    Integer.MAX_VALUE : MAX_ARRAY_SIZE;
}
```

1. 扩容的规则并不是翻倍，是原来容量⼤⼩ + 容量⼤⼩的⼀半，直⽩来说，扩容后的⼤⼩是原来容量 的 1.5 倍；
2. ArrayList 中的数组的最⼤值是 Integer.MAX_VALUE，超过这个值，JVM 就不会给数组分配内存空 间了。
3. 新增时，并没有对值进⾏严格的校验，所以 ArrayList 是允许 null 值的。

**这段源码值得我们借鉴的是**：`在进行扩容的时候，要有数组大小溢出的意识，初始判断是否大于0，还必须有一个最大限制，比如ArrayList的最大不能超过Integer.MAX_VALUE`

## 删除

```java
    public boolean remove(Object o) {
        if (o == null) {
            for (int index = 0; index < size; index++)
                if (elementData[index] == null) {
                    fastRemove(index);
                    return true;
                }
        } else {
            for (int index = 0; index < size; index++)
                if (o.equals(elementData[index])) {
                    fastRemove(index);
                    return true;
                }
        }
        return false;
    }

     /*
        快速删除
     */
    private void fastRemove(int index) {
        modCount++;
      // 减一是为了移动到被删除元素的前面
        int numMoved = size - index - 1;
        if (numMoved > 0)
          	// 进行数组拷贝，也就是覆盖
            System.arraycopy(elementData, index+1, elementData, index,
                             numMoved);
        elementData[--size] = null; // clear to let GC do its work
    }
```

如果是按照`元素`来删除，会先判断是否是`null`,找到对对应的下标。

![](https://img.mukewang.com/5d5fc643000142a403600240.gif)

## 迭代器

如果要自己实现迭代器,实现 java.util.Iterator 类就好了。ArrayList也是一样。

```java
    public Iterator<E> iterator() {
        return new Itr();
    }
```

迭代器的具体实现：

```java
private class Itr implements Iterator<E> {
        int cursor;       // 迭代过程中下一个元素的位置
        int lastRet = -1; // 新增场景:表示上一次迭代过程中,索引的位置;删除场景:为 -1。
        int expectedModCount = modCount; // 迭代过程中的期望版本好。防止迭代过程中元素新增或删除。

        Itr() {}
			// 是否有下一个元素，直接判断是否等于size就行，因为cursor指向的是下一个元素的位置
        public boolean hasNext() {
            return cursor != size;
        }

        @SuppressWarnings("unchecked")
        public E next() {
          // 检查是否被修改过。
            checkForComodification();
          	// 
            int i = cursor;
            if (i >= size)
                throw new NoSuchElementException();
            Object[] elementData = ArrayList.this.elementData;
          // 判断是否数组下标越界
            if (i >= elementData.length)
                throw new ConcurrentModificationException();
            cursor = i + 1;
            return (E) elementData[lastRet = i];
        }

        public void remove() {
          // 防止对同一个元素重复调用。
            if (lastRet < 0)
                throw new IllegalStateException();
            checkForComodification();

            try {
                ArrayList.this.remove(lastRet);
                cursor = lastRet;
                lastRet = -1;
                expectedModCount = modCount;
            } catch (IndexOutOfBoundsException ex) {
                throw new ConcurrentModificationException();
            }
        }

        @Override
        @SuppressWarnings("unchecked")
        public void forEachRemaining(Consumer<? super E> consumer) {
            Objects.requireNonNull(consumer);
            final int size = ArrayList.this.size;
            int i = cursor;
            if (i >= size) {
                return;
            }
            final Object[] elementData = ArrayList.this.elementData;
            if (i >= elementData.length) {
                throw new ConcurrentModificationException();
            }
            while (i != size && modCount == expectedModCount) {
                consumer.accept((E) elementData[i++]);
            }
            // update once at end of iteration to reduce heap write traffic
            cursor = i;
            lastRet = i - 1;
            checkForComodification();
        }

        final void checkForComodification() {
            if (modCount != expectedModCount)
                throw new ConcurrentModificationException();
        }
    }
```

`注意`：lastRet = -1 的操作目的,是防止重复删除操作

# 时间复杂度

从我们上面新增或删除方法的源码解析,对数组元素的操作,只需要根据数组索引,直接新增或删除，所以时间复杂度为o(1)。

# 线程安全

ArrayList作为共享变量时，才会有线程安全问题，因为里面的`elementData`、`size`并没有使用`volatile`进行修饰，所有有可见性问题，同时`add`、`remove`等方法并不是同步方法，所有还会有原子性问题。