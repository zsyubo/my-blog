# 概览

SynchronousQueue 是比较独特的队列，其本身是没有容量大小，`比如我放一个数据到队列中，我是不能够立马返回的，我必须等待别人把我放进去的数据消费掉了，才能够返回`。SynchronousQueue 在消息队列技术中间件中被大量使用。

**继承结构**

SynchronousQueue 整体类图和 LinkedBlockingQueue 相似，都是实现了 BlockingQueue 接口，但因为其不储存数据结构，有一些方法是没有实现的，比如说 isEmpty、size、contains、remove 和迭代等方法，这些方法都是默认实现。

**底层数据结构**

```java
// 堆栈和队列共同的接口
    // 负责执行 put or take
    abstract static class Transferer<E> {
        // e 为空的，会直接返回特殊值，不为空会传递给消费者
        // timed 为 true，说明会有超时时间
        abstract E transfer(E e, boolean timed, long nanos);
    }

    // 堆栈 后入先出 非公平(默认)
    // Scherer-Scott 算法
    static final class TransferStack<E> extends Transferer<E> {
    }

    // 队列 先入先出 公平
    static final class TransferQueue<E> extends Transferer<E> {
    }

    private transient volatile Transferer<E> transferer;

    // 无参构造器默认为非公平的
    public SynchronousQueue(boolean fair) {
        transferer = fair ? new TransferQueue<E>() : new TransferStack<E>();
    }
```

1. 堆栈和队列都有一个共同的接口，叫做 Transferer，该接口有个方法：transfer，该方法很神奇，会承担 take 和 put 的双重功能；
2. 在我们初始化的时候，是可以选择是使用堆栈还是队列的，如果你不选择，默认的就是堆栈，类注释中也说明了这一点，堆栈的效率比队列更高。

# 源码

## 非公平的堆栈

默认都是非公平的方式，这里的非公平的意思是先put，不一定先消费(先入先出)。

**入栈、出栈**

大致过程

1. 判断是 put 方法还是 take 方法；
2. 判断栈头数据是否为空，如果为空或者栈头的操作和本次操作一致，是的话走 3，否则走 5；
3. 判断操作有无设置超时时间，如果设置了超时时间并且已经超时，返回 null，否则走 4；
4. 如果栈头为空，把当前操作设置成栈头，或者栈头不为空，但栈头的操作和本次操作相同，也把当前操作设置成栈头，并看看其它线程能否满足自己，不能满足则阻塞自己。比如当前操作是 take，但队列中没有数据，则阻塞自己；
5. 如果栈头已经是阻塞住的，需要别人唤醒的，判断当前操作能否唤醒栈头，可以唤醒走 6，否则走 4；
6. 把自己当作一个节点，赋值到栈头的 match 属性上，并唤醒栈头节点；
7. 栈头被唤醒后，拿到 match 属性，就是把自己唤醒的节点的信息，返回。

```java
// transfer 方法思路比较复杂，因为 take 和 put 两个方法都揉在了一起
@SuppressWarnings("unchecked")
E transfer(E e, boolean timed, long nanos) {
    SNode s = null; // constructed/reused as needed
    // e 为空，说明是 take 方法，不为空是 put 方法
    int mode = (e == null) ? REQUEST : DATA;
    // 自旋
    for (;;) {
        // 拿出头节点，有几种情况
        // 1：头节点为空，说明队列中还没有数据
        // 2：头节点不为空，并且是 take 类型的，说明头节点线程正等着拿数据。
        // 3：头节点不为空，并且是 put 类型的，说明头节点线程正等着放数据。
        SNode h = head;
        // 栈头为空，说明队列中还没有数据。
        // 栈头不为空，并且栈头的类型和本次操作一致，比如都是 put，那么就把
        // 本次 put 操作放到该栈头的前面即可，让本次 put 能够先执行
        if (h == null || h.mode == mode) {  // empty or same-mode
            // 设置了超时时间，并且 e 进栈或者出栈要超时了，
            // 就会丢弃本次操作，返回 null 值。
            // 如果栈头此时被取消了，丢弃栈头，取下一个节点继续消费
            if (timed && nanos <= 0) {      // can't wait
                // 栈头操作被取消
                if (h != null && h.isCancelled())
                    // 丢弃栈头，把栈头后一个元素作为栈头
                    casHead(h, h.next);     // pop cancelled node
                //栈头是空的，直接返回 null
                else
                    return null;
            // 没有超时，直接把 e 作为新的栈头
            } else if (casHead(h, s = snode(s, e, h, mode))) {
                // e 等待出栈，一种是空队列 take，一种是 put
                SNode m = awaitFulfill(s, timed, nanos);
                if (m == s) {               // wait was cancelled
                    clean(s);
                    return null;
                }
                // 本来 s 是栈头的，现在 s 不是栈头了，s 后面又来了一个数，把新的数据作为栈头
                if ((h = head) != null && h.next == s)
                    casHead(h, s.next);     // help s's fulfiller
                return (E) ((mode == REQUEST) ? m.item : s.item);
            }
        // 栈头正在等待其他线程 put 或 take
        // 比如栈头正在阻塞，并且是 put 类型，而此次操作正好是 take 类型，走此处
        } else if (!isFulfilling(h.mode)) { // try to fulfill
            // 栈头已经被取消，把下一个元素作为栈头
            if (h.isCancelled())            // already cancelled
                casHead(h, h.next);         // pop and retry
            // snode 方法第三个参数 h 代表栈头，赋值给 s 的 next 属性
            else if (casHead(h, s=snode(s, e, h, FULFILLING|mode))) {
                for (;;) { // loop until matched or waiters disappear
                    // m 就是栈头，通过上面 snode 方法刚刚赋值
                    SNode m = s.next;       // m is s's match
                    if (m == null) {        // all waiters are gone
                        casHead(s, null);   // pop fulfill node
                        s = null;           // use new node next time
                        break;              // restart main loop
                    }
                    SNode mn = m.next;
                     // tryMatch 非常重要的方法，两个作用：
                     // 1 唤醒被阻塞的栈头 m，2 把当前节点 s 赋值给 m 的 match 属性
                     // 这样栈头 m 被唤醒时，就能从 m.match 中得到本次操作 s
                     // 其中 s.item 记录着本次的操作节点，也就是记录本次操作的数据
                    if (m.tryMatch(s)) {
                        casHead(s, mn);     // pop both s and m
                        return (E) ((mode == REQUEST) ? m.item : s.item);
                    } else                  // lost match
                        s.casNext(m, mn);   // help unlink
                }
            }
        } else {                            // help a fulfiller
            SNode m = h.next;               // m is h's match
            if (m == null)                  // waiter is gone
                casHead(h, null);           // pop fulfilling node
            else {
                SNode mn = m.next;
                if (m.tryMatch(h))          // help match
                    casHead(h, mn);         // pop both h and m
                else                        // lost match
                    h.casNext(m, mn);       // help unlink
            }
        }
    }
}
```

