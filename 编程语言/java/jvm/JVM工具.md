# 1. jps命令

>    jps(Java Virtual Machine Process Status Tool)是JDK 1.5提供的一个显示当前所有java进程pid的命令，简单实用，非常适合在linux/unix平台上简单察看当前java进程的一些简单情况。
>
>    ![](https://s1.ax1x.com/2020/04/08/GRZPU0.png)

参考url：https://www.jianshu.com/p/d39b2e208e72

注：jps 支持查看远程服务上的 jvm 进程信息。如果需要查看其他机器上的 jvm 进程，需要在待查看机器上启动 jstatd 服务。

#### 参数说明

1. -q：只输出进程 ID
2. -m：输出传入 main 方法的参数
3. -l：输出完全的包名，应用主类名，jar的完全路径名
4. -v：输出jvm参数
5. -V：输出通过flag文件传递到JVM中的参数



# 2. jinfo命令

`注`:参考自https://blog.csdn.net/winwill2012/article/details/46336839

>   jinfo可以用来查看正在运行的java运用程序的扩展参数，甚至支持在运行时动态地更改部分参数，他的基本使用语法如下：
>   jinfo -< option > < pid >
>
>   <img src="https://s1.ax1x.com/2020/04/08/GRZi5V.png" style="zoom:75%;" />


#### 参数说明

1. -flag< name >  jvm进程pid: 打印指定java虚拟机的参数值。 
2. -flag [+|-]< name > jvm进程pid：设置或取消指定java虚拟机参数的布尔值。 
3. -flag < name >=< value > jvm进程pid：设置指定java虚拟机的参数的值。

3. -flags  jvm进程pid：查看jvm的启动运行中手动设值的参数。

#### 使用示例

1. 下面的命令显示了新生代对象晋升到老年代对象的最大年龄。在运行程序运行时并没有指定这个参数，但是通过jinfo，可以查看这个参数的当前的值。 

![](https://s1.ax1x.com/2020/04/08/GRZE2F.png)

2. 下面的命令显示是否打印gc详细信息： 

   ![](https://s1.ax1x.com/2020/04/08/GRZeKJ.png)


3. 下面的命令在运用程序运行时动态打开打印详细gc信息开关： 

   ![](https://s1.ax1x.com/2020/04/08/GRZnbR.png)

4. 查看最大内存

   ![](https://s1.ax1x.com/2020/04/08/GRZKV1.png)

5. 查看jvm手动设值的参数 

   ![](https://s1.ax1x.com/2020/04/08/GRZ1PK.png)

`注意`:jinfo虽然可以在java程序运行时动态地修改虚拟机参数，但并不是所有的参数都支持动态修改。



# 3.jstat命令

参考文章：https://www.cnblogs.com/boothsun/p/8127552.html | https://www.cnblogs.com/parryyang/p/5772484.html |  https://www.cnblogs.com/myna/p/7567769.html

> jstat是一个用来监控虚拟机资源和性能的命令行工具。 可以展示本机或者远程虚拟机进程中的类装载、内存、垃圾回收、JIT编译等运行数据，是常见的线上jvm问题排查的工具，非常实用。



命令格式：`jstat [options] VMID [interval] [count]`  , [interval]指时间间隔， [count]一共输出10次结果

#### 参数详解

| Option           | Displays                                                     |
| ---------------- | ------------------------------------------------------------ |
| class            | 类加载的行为统计。                                           |
| compiler         | HotSpt JIT编译器行为统计。                                   |
| gc               | 垃圾回收堆的行为统计。                                       |
| gccapacity       | 各个垃圾回收代容量(young,old,perm)和他们相应的空间统计。     |
| gcutil           | 垃圾回收统计概述（百分比）。                                 |
| gccause          | 垃圾收集统计概述（同-gcutil），附加最近两次垃圾回收事件的原因。S |
| gcnew            | 新生代行为统计。                                             |
| gcnewcapacity    | 新生代与其相应的内存空间的统计。                             |
| gcold            | 年老代和永生代行为统计。                                     |
| gcoldcapacity    | 年老代行为统计。                                             |
| gcpermcapacity   | 永生代行为统计。                                             |
| printcompilation | HotSpot编译方法统计。                                        |

#### 使用示例
1. 查询类加载信息，并且每隔10秒加载一次，总共10次。

   ![](https://s1.ax1x.com/2020/04/08/GRZYKH.png)

> Loaded 加载类的数量
> Bytes 加载类合计大小
> Unloaded 卸载类的数量
> Bytes 卸载类合计大小
> Time 表示加载和卸载类总共的耗时

2. 查看GC信息

   ![](https://s1.ax1x.com/2020/04/08/GRZtrd.png)

> `C即Capacity 总容量，U即Used 已使用的容量`
> 参数	描述
> NGCMN	新生代最小容量
> NGCMX	新生代最大容量
> NGC	当前新生代容量
> S0C	第一个幸存区大小
> S1C	第二个幸存区的大小
> EC	伊甸园区的大小
> OGCMN	老年代最小容量
> OGCMX	老年代最大容量
> OGC	当前老年代大小
> OC	当前老年代大小
> MCMN	最小元数据容量
> MCMX	最大元数据容量
> MC	当前元数据空间大小
> CCSMN	最小压缩类空间大小
> CCSMX	最大压缩类空间大小
> CCSC	当前压缩类空间大小
> YGC	年轻代gc次数
> FGC	老年代GC次数

3. 输出JIT编译过的方法数量耗时等

   ![](https://s1.ax1x.com/2020/04/08/GRZaVI.png)

> Compiled : 编译数量
> Failed : 编译失败数量
> Invalid : 无效数量
> Time : 编译耗时
> FailedType : 失败类型
> FailedMethod : 失败方法的全限定名

#  5.  jstack
> jstack是java虚拟机自带的一种堆栈跟踪工具。jstack用于打印出给定的java进程ID或core file或远程调试服务的Java堆栈信息

<img src="https://s1.ax1x.com/2020/04/08/GRZdat.png" style="zoom:67%;" />