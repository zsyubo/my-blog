# 概览

LinkedList是集合类List的另一种实现，底层数据结构是一个双向链表，Linked特点是适用于先入后出、先入先出的等情况，所以在对列源码中多次被频繁使用。

![](https://s2.ax1x.com/2020/01/16/ljEFZF.png)

- 链表每个节点我们叫做 Node，Node 有 prev 属性，代表前⼀个节点的位置，next 属性，

- 代 表后⼀个节点的位置； first 是双向链表的头节点，它的前⼀个节点是 null。 
- last 是双向链表的尾节点，它的后⼀个节点是 null； 
- 当链表中没有数据时，first 和 last 是同⼀个节点，前后指向都是 null； 
- 因为是个双向链表，只要机器内存⾜够强⼤，是没有⼤⼩限制的。

**Node**

```
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

## 增加

追加节点可以默认从尾部增加(add)，但是也可以从头部追加数据(addFirst)。

**尾部增加节点**

```
    public boolean add(E e) {
        linkLast(e);
        return true;
    }
```

`linkLast`

```
    void linkLast(E e) {
    		// 尾节点暂存
        final Node<E> l = last;
        // 
        final Node<E> newNode = new Node<>(l, e, null);
        last = newNode;
        if (l == null)
            first = newNode;
        else
            l.next = newNode;
        size++;
        modCount++;
    }
```

