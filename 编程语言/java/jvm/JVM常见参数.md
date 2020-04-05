###读懂虚拟机日志


# 设置Java虚拟机参数
> java [-options] class [args...]

`-options`表示Java虚拟机的启动参数，`class`为带有`main()`函数的Java类，`args`表示传递给主函数`main()`的参数,在指定虚拟机参数时有如下3中方式，但设置堆大小等参数不包括其中。
1. `-XX:+<option>` 开启option参数
2. `-XX:-<option>` 关闭option参数
3. `-XX:<option>=<value>` 将option参数的值设置为value

#### 基本参数
1. `-XX:+PrintGC`：当虚拟机发生GC时，打印日志。

2. `-XX:+PrintGCDetails`：输出更加详细的信息。同时会使虚拟机在退出前打印堆得详细信息。

3. `-XX:+PrintHeapAtGC`: 会在每次GC前后分别打印堆得信息。

4. `-XX:+PrintGCTimeStamps`:该参数会在每次发生GC时，额外输出GC的发生时间（GC日志头部），该时间为虚拟机启动后的时间偏移量。

5. `-XX:+PrintGCApplicationConcurrentTime`：打印应用程序的执行时间

6. `-XX:+PrintGCApplicationStoppedTime`：打印应用程序因为GC而产生的停顿时间。

7. `-XX:+PrintReferenceGC`: 跟踪系统类的软引用、弱引用和Finallize队列。

8. `-Xloggc:/Users/zsyubo/Downloads/gc.log`: 可以在指定目录下的gc.log文件中记录所有的GC日志。



#### 类加载/卸载的跟踪

1. `-verbose:class`：跟踪类的加载和卸载。

2. `-XX:+TraceClassLoading`:只跟踪类的加载

3. `-XX:+TraceClassUnloading`:只跟踪类的卸载



4. `-XX:+PrintClassHistogram`:在程序运行时打印、查看系统中的类的分布情况，当需要时在控制台按下Ctrl+Break组合键，控制台就会显示这些信息。



#### 系统参数查看
1. `-XX:+PrintVMOptions`:在程序运行时，打印虚拟机接收到的命令行显示参数。
2. `-XX:+PrintCommandLineFlags` ：可以打印传递给虚拟机的显示和隐式参数，隐式参数可能是虚拟机启动时自行设置的。
3. `-XX:+PrintFlagsFinal`:打印所有的系统参数的值。

##堆配置参数
当Java进程启动时，虚拟机会分配一块初始堆空间。一般来说，虚拟机会尽可能维持在初始堆空间的范围内运行。但是如果初始堆空间耗尽，虚拟机会对堆空间进行扩展，其扩展上限为最大对空间。
1. `-Xms`:指定堆空间初始大小，例：`-Xms6m`
2. `-Xmx`：指定堆空间最大多少，例：`-Xmx16m`
实际工作中，可以直接将对`-Xms`和`-Xmx`设置相等。这样好处是可以减少程序运行时进行垃圾回收的次数，从而提高程序性能。
####新生代配置参数 注：`新生代=eden+from+to`
1. `-Xmn`: 设置新生代大小。设置一个较大的新生代会减少老年代的大小。新生代的大小一般设置为整个堆空间的1/3到1/4左右。
2. `-XX:SurvivorRatio`:用来设置新生代中eden空间和from/to空间的比例关系。它的含义如下：-XX:SurvivorRatio=eden/from=eden/to
尽可能将对象预留在新生代，减少老年代GC的次数
3. `-XX:NewRatio=老年代/新生代` ：老年代与新生代的比例
当from to区域不足以装载从eden gc的对象时，可能会向老年代进行担保。



#### 堆溢出处理
在程序运行中，如果堆空间不足，则有可能抛出内存溢出错误（Out of Memory），简称OOM
1. `-XX:+HeapDumpOnOutOfMemoryError`:此参数可以在内存溢出时导出整个堆信息。
2. `-XX:HeapDumpPath=保存dump路径+保存的文件名.dump`:此参数与上面参数结合使用，指定导出堆打的存放路径。
3. `-XX:OnOutOfMemoryError=执行脚本路径`：当虚拟机发生错误时，执行此脚本文件。



## 非堆内存的参数配置
非堆内存只要是一些用于方法区、线程栈和执行内存的内存。他们与堆内存相互对立。
##### 方法区配置
1. JDK1.6/JDK1.7 :`-XX:PermSize`（初始的永久区大小）和`-XX:MaxPermSize`（最大永久区大小）指定永久带的大小
2. 1.8中，永久区被彻底删除，使用了新的元数据区存放类的元数据。默认情况下，元数据区只受系统可用内存的限制，但是可以使用参数`-XX:MaxMetaspaceSize`指定永久区的最大可用值。
##### 栈配置
1. `-Xss`：指定线程的栈大小

##### 直接内存配置
1. `-XX:MaxDirectMemorySize`: 设置最大可用直接内存，如不设置，默认值为最大堆空间。当直接内存使用量达到配置值是，就会触发垃圾回收，如果不能释放足够空间，直接内存溢出引发系统的OOM。
2. 直接内存适合申请次数较少、访问较频繁的场合。如果内存空间需要频繁申请，则并不适合使用直接内存。