# 概述

DelayQueue是一个延时队列。简单说，延时实现是使用的锁，如果一个延迟5秒执行，那么当前线程就会沉睡5秒。底层是使用的数组来存放延时元素。

## 继承结构

```java
public class DelayQueue<E extends Delayed> extends AbstractQueue<E>
    implements BlockingQueue<E> {
      private final transient ReentrantLock lock = new ReentrantLock();
```

从泛型中可以看出，DelayQueue 中的元素必须是 Delayed 的子类，Delayed 是表达延迟能力的关键接口，其继承了 Comparable 接口，并定义了还剩多久过期的方法，如下：

```java
public interface Delayed extends Comparable<Delayed> {
    long getDelay(TimeUnit unit);
}
```



# 源码

## put

```java
public void put(E e) {
     offer(e);
}
public boolean offer(E e) {
    final ReentrantLock lock = this.lock;
    // 上锁
    lock.lock();
    try {
        // 使用 PriorityQueue 的扩容，排序等能力
        q.offer(e);
        // 如果恰好刚放进去的元素正好在队列头
        // 立马唤醒 take 的阻塞线程，执行 take 操作
        // 如果元素需要延迟执行的话，可以使其更快的沉睡计时
        if (q.peek() == e) {
            leader = null;
            available.signal();
        }
        return true;
    } finally {
        // 释放锁
        lock.unlock();
    }
}

// 新增元素
public boolean offer(E e) {
    // 如果是空元素的话，抛异常
    if (e == null)
        throw new NullPointerException();
    modCount++;
    int i = size;
    // 队列实际大小大于容量时，进行扩容
    // 扩容策略是：如果老容量小于 64，2 倍扩容，如果大于 64，50 % 扩容
    if (i >= queue.length)
        grow(i + 1);
    size = i + 1;
    // 如果队列为空，当前元素正好处于队头
    if (i == 0)
        queue[0] = e;
    else
    // 如果队列不为空，需要根据优先级进行排序
        siftUp(i, e);
    return true;
}
   // 按照从小到大的顺序排列
    private void siftUpComparable(int k, E x) {
        Comparable<? super E> key = (Comparable<? super E>) x;
        // k 是当前队列实际大小的位置
        while (k > 0) {
            // 对 k 进行减倍
            int parent = (k - 1) >>> 1;
            Object e = queue[parent];
            // 如果 x 比 e 大，退出，把 x 放在 k 位置上
            if (key.compareTo((E) e) >= 0)
                break;
            // x 比 e 小，继续循环，直到找到 x 比队列中元素大的位置
            queue[k] = e;
            k = parent;
        }
        queue[k] = key;
}

/**
扩容
**/
    private void grow(int minCapacity) {
        int oldCapacity = queue.length;
        // Double size if small; else grow by 50%
        int newCapacity = oldCapacity + ((oldCapacity < 64) ?
                                         (oldCapacity + 2) :
                                         (oldCapacity >> 1));
        // overflow-conscious code
        if (newCapacity - MAX_ARRAY_SIZE > 0)
            newCapacity = hugeCapacity(minCapacity);
        queue = Arrays.copyOf(queue, newCapacity);
    }
```

1. 对新增元素进行判空；
2. 对队列进行扩容，扩容策略和集合的扩容策略很相近；
3. 根据元素的 compareTo 方法进行排序，我们希望最终排序的结果是从小到大的，因为我们想让队头的都是过期的数据，我们需要在 compareTo 方法里面实现：通过每个元素的过期时间进行排序，如下：

## take

为了保证线程安全，take一开始就加锁了。

```java
    public E take() throws InterruptedException {
      	// 加锁
        final ReentrantLock lock = this.lock;
        lock.lockInterruptibly();
        try {
            for (;;) {
                // 从队头中拿数据出来
                E first = q.peek();
                // 如果为空，说明队列中，没有数据，阻塞住
                if (first == null)
                    available.await();
                else {
                    // 获取队头数据的过期时间
                    long delay = first.getDelay(NANOSECONDS);
                    // 如果过期了，直接返回队头数据
                    if (delay <= 0)
                        return q.poll();
                    // 引用置为 null ，便于 gc，这样可以让线程等待时，回收 first 变量
                    first = null;
                    // leader 不为空的话，表示当前队列元素之前已经被设置过阻塞时间了
                    // 直接阻塞当前线程等待。
                    if (leader != null)
                        available.await();
                    else {
                      // 之前没有设置过阻塞时间，按照一定的时间进行阻塞
                        Thread thisThread = Thread.currentThread();
                        leader = thisThread;
                        try {
                            // 进行阻塞
                            available.awaitNanos(delay);
                        } finally {
                            if (leader == thisThread)
                                leader = null;
                        }
                    }
                }
            }
          } finally {
            if (leader == null && q.peek() != null)
                available.signal();
            lock.unlock();
        }
    }
```

# 总结

DelayQueue的原理并不算复杂，但是很有意思。

虽然put和take是互斥的，但是，take在获取第一个元素时，如果还没到延时时间。直接让此线程睡眠第一个元素的延时时间，保证资源的不浪费。