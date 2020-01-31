# 概览

LinkedList是集合类List的另一种实现，底层数据结构是一个双向链表，Linked特点是适用于先入后出、先入先出的等情况，所以在对列源码中多次被频繁使用。

![](https://s2.ax1x.com/2020/01/16/ljEFZF.png)

- 链表每个节点我们叫做 Node，Node 有 prev 属性，代表前⼀个节点的位置，next 属性，

- 代 表后⼀个节点的位置； first 是双向链表的头节点，它的前⼀个节点是 null。 
- last 是双向链表的尾节点，它的后⼀个节点是 null； 
- `当链表中没有数据时，first 和 last 是同⼀个节点，前后指向都是 null`； 
- 因为是个双向链表，只要机器内存⾜够强⼤，是没有⼤⼩限制的。

**Node**

```java
 private static class Node<E> {
        E item; // 节点值
        Node<E> next; // 指向下一个节点
        Node<E> prev; // 指向前一个节点
				// 初始化
        Node(Node<E> prev, E element, Node<E> next) {
            this.item = element;
            this.next = next;
            this.prev = prev;
        }
    }
```

# 源码解析

## 构造方法

```java
// 默认构造方法   
public LinkedList() {
}

public LinkedList(Collection<? extends E> c) {
     this();
     addAll(c);
}
```



## 增加

追加节点可以默认从尾部增加(add)，但是也可以从头部追加数据(addFirst)。

可以插入null

**尾部增加节点**

```java
    public boolean add(E e) {
        linkLast(e);
        return true;
    }
```

`linkLast`

```java
    void linkLast(E e) {
    		// 尾节点暂存
        final Node<E> l = last;
        // 构造新node
        final Node<E> newNode = new Node<>(l, e, null);
      // 设置首尾
        last = newNode;
      // 如果首部为空，那么新node就是first
        if (l == null)
            first = newNode;
        else
            l.next = newNode;
        size++;
        modCount++;
    }
```

从头部添加

```java
    private void linkFirst(E e) {
        final Node<E> f = first;
        final Node<E> newNode = new Node<>(null, e, f);
        first = newNode;
        if (f == null)
            last = newNode;
        else
            f.prev = newNode;
        size++;
        modCount++;
    }

```

## 删除

```java
    public boolean remove(Object o) {
        if (o == null) {
            for (Node<E> x = first; x != null; x = x.next) {
                if (x.item == null) {
                    unlink(x);
                    return true;
                }
            }
        } else {
            for (Node<E> x = first; x != null; x = x.next) {
                if (o.equals(x.item)) {
                    unlink(x);
                    return true;
                }
            }
        }
        return false;
    }
```

`unlink`：从头删除节点 f 是链表头节点

```java
E unlink(Node<E> x) {
         // 拿出节点的值，作为方法的返回值
        final E element = x.item;
        // 拿出头节点的下一个节点
        final Node<E> next = x.next;
        // 拿出上一个节点
        final Node<E> prev = x.prev;
  			
        if (prev == null) {
          // 上一个节点为null则当前节点为首节点
            first = next;
        } else {
            prev.next = next;
            x.prev = null;
        }

        if (next == null) {
            last = prev;
        } else {
            next.prev = prev;
            x.next = null;
        }

        x.item = null;
        size--;
        modCount++;
        return element;
    }
```

## 查找

`根据index查询node`:采用了二分法。但是其他非index查找node都是暴力for循环。

```java
   Node<E> node(int index) {
        // assert isElementIndex(index);
            // 如果数据位于 前半部分
        if (index < (size >> 1)) {
            Node<E> x = first;
            for (int i = 0; i < index; i++)
                x = x.next;
            return x;
        } else {
            // 如果数据位于 后半部分
            Node<E> x = last;
            for (int i = size - 1; i > index; i--)
                x = x.prev;
            return x;
        }
    }
```

# 其他

| 方法含义 | 返回异常  | 返回特殊值 | 底层实现                                    |
| -------- | --------- | ---------- | ------------------------------------------- |
| 新增     | add(e)    | offer(e)   | 底层实现相同                                |
| 删除     | remove(e) | poll(e)    | 链表为空时，remove会抛出异常，poll返回null  |
| 查找     | element() | peek()     | 链表为空时，element会抛出异常，peek返回null |

