# 简介

<img src="https://s2.ax1x.com/2020/01/06/lrwVhD.png" style="zoom:50%;" />

# 流程

1. 客户端调用`close`方法，执行主动关闭，这时候`客户端`就不能继续发送数据给`服务端`了， 这包其实就是tcp头部的`fin`字段置为了1。同时这个fin状态相当于一个数据，所有需要消耗序列号，同时也需要对方ack确认的。

   **注意**：FIN 段是可以携带数据的，比如客户端可以在它最后要发送的数据块可以“捎带” FIN 段。当然也可以不携带数据。不管 FIN 段是否携带数据，都需要消耗一个序列号。



# 4次可以变三次么？

当然可以，因为有**延迟确认**的存在，把第二步的 ACK 经常会跟随第三步的 FIN 包一起捎带会对端。延迟确认后面有一节专门介绍。

<img src="https://s2.ax1x.com/2020/01/07/lyO1Og.png" style="zoom:50%;" />

# 同时关闭

![](https://s2.ax1x.com/2020/01/07/lyO06U.png)

- 最初客户端和服务端都处于 ESTABLISHED 状态
- 客户端发送 `FIN` 包，等待对端对这个 FIN 包的 ACK，随后进入 `FIN-WAIT-1` 状态
- 处于`FIN-WAIT-1`状态的客户端还没有等到 ACK，收到了服务端发过来的 FIN 包
- 收到 FIN 包以后客户端会发送对这个 FIN 包的的确认 ACK 包，同时自己进入 `CLOSING` 状态
- 继续等自己 FIN 包的 ACK
- 处于 `CLOSING` 状态的客户端终于等到了ACK，随后进入`TIME-WAIT`
- 在`TIME-WAIT`状态持续 2*MSL，进入`CLOSED`状态