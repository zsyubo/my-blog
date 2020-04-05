# 启

要打印出来需要指定`-XX:+PrintGCDetails`。

下面为GC案例：
`[GC (System.gc()) [PSYoungGen: 4096K->0K(9216K)] 8576K->8576K(19456K), 0.0022658 secs] [Times: user=0.00 sys=0.00, real=0.01 secs] `
`[Full GC (System.gc()) [PSYoungGen: 0K->0K(9216K)] [ParOldGen: 8576K->4480K(10240K)] 8576K->4480K(19456K), [Metaspace: 3271K->3271K(1056768K)], 0.0205769 secs] [Times: user=0.02 sys=0.00, real=0.02 secs] `

![](https://s1.ax1x.com/2020/03/30/Ged75D.png)

![](https://s1.ax1x.com/2020/03/30/Gedq8H.png)

# GC类型

区域名称与GC类型对应（老年代与永久代同理）：

| GC区域名称 |               对应收集器                | 收集区域 |
| ---------- | :-------------------------------------: | :------: |
| DefNes     | Serial 收集器（Default New Generation） |  新生代  |
| ParNew     |  ParNew收集器（Parallel New Generation  |  新生代  |
| PSYoungGen |         Parallel Scavenge收集器         |  新生代  |
注:CMS和G1日志不用串行/并行收集器日志一样。

# GC原因

| 代码               | 原因             |
| ------------------ | ---------------- |
| System.gc()        | System.gc();调用 |
| Allocation Failure | 分配对象失败     |

# 堆内存日志

```
Heap
 par new generation   total 4608K, used 2508K [0x00000007bf600000, 0x00000007bfb00000, 0x00000007bfb00000)
  eden space 4096K,  51% used [0x00000007bf600000, 0x00000007bf811470, 0x00000007bfa00000)
  from space 512K,  76% used [0x00000007bfa00000, 0x00000007bfa61c98, 0x00000007bfa80000)
  to   space 512K,   0% used [0x00000007bfa80000, 0x00000007bfa80000, 0x00000007bfb00000)
 concurrent mark-sweep generation total 5120K, used 1713K [0x00000007bfb00000, 0x00000007c0000000, 0x00000007c0000000)
 Metaspace       used 2987K, capacity 4496K, committed 4864K, reserved 1056768K
  class space    used 326K, capacity 388K, committed 512K, reserved 1048576K
```

`par new generation`新生代总共有`4.5m`可用内存，已使用`2508K`

`concurrent mark-sweep`CMS收集器管理的老年代内存空间一共是5mb。



