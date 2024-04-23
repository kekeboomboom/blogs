# JVM调优



## 什么时候JVM调优



要对Java应用程序进行调优，优化JVM并不是第一选择。我们首先应该考虑软件架构和代码优化等方面，这方面的优化可能会取得更大的进步空间。因此假设我们已经对于软件架构、代码优化、数据库优化等等做过了一些努力，接着我们希望通过JVM调优来做一些事情，那么我们可以接着往下读。

性能优化的一些方式：

![1](https://yqintl.alicdn.com/17cc09a576b8541c03bea09c5a9bb0b35d14a6ec.png)

## JVM调优指标

我们对JVM调优有哪些指标呢？一般来说有下面三点：

- 吞吐量(**Throughput**)：is the percentage of time the VM spends executing the application versus time spent performing garbage collection.
- 延时(**Latency**)：is the amount of time required to run a garbage collection event.
- 资源占用(**Footprint**)：is the amount of memory required by the garbage collector to run smoothly.

如果能增加资源投入，提高CPU、内存等，自然可以提高吞吐量和减少延时。

对于吞吐量和延时，我们一般通过调节垃圾收集参数来做权衡。而对于吞吐量和延时的不同的统计方式，可能会得到不同的结果。

对于垃圾收集对应用程序请求的影响的计算方法，可以参考[美团文章](https://tech.meituan.com/2017/12/29/jvm-optimize.html)。通过统计一分钟内请求受影响的占比，来判断GC影响时间是否减少。

我们还可以开启GC日志，来看每次垃圾收集的时间、频率，来判断GC总时间是否减少。

当我们进行各种压力测试，基准测试后，拿到这个测试数据，才能判断是否达到了我们预设的指标。

## 获取JVM监控数据

### 开启GC log

```
-XX:+PrintGC
-XX:+PrintGCTimeStamps 
-XX:+PrintGCDetails 
-Xloggc:<filename>
```

- -Xloggc specifies where the file is located
- -XX:+PrintGCDetails – includes additional details in the garbage collector log
- -XX:+PrintGCTimeStamps – prints the timestamps to the log

```
0.134: [GC (Allocation Failure) [PSYoungGen: 65536K->10720K(76288K)] 65536K->40488K(251392K), 0.0190287 secs] [Times: user=0.13 sys=0.04, real=0.02 secs]
0.193: [GC (Allocation Failure) [PSYoungGen: 71912K->10752K(141824K)] 101680K->101012K(316928K), 0.0357512 secs] [Times: user=0.27 sys=0.06, real=0.04 secs]
0.374: [GC (Allocation Failure) [PSYoungGen: 141824K->10752K(141824K)] 232084K->224396K(359424K), 0.0809666 secs] [Times: user=0.58 sys=0.12, real=0.08 secs]
0.455: [Full GC (Ergonomics) [PSYoungGen: 10752K->0K(141824K)] [ParOldGen: 213644K->215361K(459264K)] 224396K->215361K(601088K), [Metaspace: 2649K->2649K(1056768K)], 0.4409247 secs] [Times: user=3.46 sys=0.02, real=0.44 secs]
0.984: [GC (Allocation Failure) [PSYoungGen: 131072K->10752K(190464K)] 346433K->321225K(649728K), 0.1407158 secs] [Times: user=1.28 sys=0.08, real=0.14 secs]
1.168: [GC (System.gc()) [PSYoungGen: 60423K->10752K(190464K)] 370896K->368961K(649728K), 0.0676498 secs] [Times: user=0.53 sys=0.05, real=0.06 secs]
1.235: [Full GC (System.gc()) [PSYoungGen: 10752K->0K(190464K)] [ParOldGen: 358209K->368152K(459264K)] 368961K->368152K(649728K), [Metaspace: 2652K->2652K(1056768K)], 1.1751101 secs] [Times: user=10.64 sys=0.05, real=1.18 secs]
2.612: [Full GC (Ergonomics) [PSYoungGen: 179712K->0K(190464K)] [ParOldGen: 368152K->166769K(477184K)] 547864K->166769K(667648K), [Metaspace: 2659K->2659K(1056768K)], 0.2662589 secs] [Times: user=2.14 sys=0.00, real=0.27 secs]
```

开启GClog可得到如上日志，不同的垃圾收集器可能形式略有差异，但都大致相同。上面写了由于内存分配失败而导致full GC。显示了新生代，老年代，堆内存，元空间垃圾收集前和后的空间大小的变化。垃圾收集时间，用户态时间、内核态时间、真正用时等。



关于gclog 文件的分析，可以参考https://sematext.com/blog/java-garbage-collection-logs/#parallel-and-concurrent-mark-sweep-garbage-collectors。至于好用的免费可视化工具没有发现，如果有人知道可评论区指出。



### jmap

此命令可以获得当前堆快照，我使用JProfiler来查看堆信息。[官方操作文档](https://www.ej-technologies.com/resources/jprofiler/help/doc/heapWalker/hprofSnapshots.html)

先使用 `jps -v`查看Java程序进程id，然后使用`jmap -dump:live,format=b,file=<filename> <PID>`，filename可以起名为xxx.hprof

> -XX:+HeapDumpOnOutOfMemoryError -XX:HeapDumpPath=./heapDump.hprof 此两个参数当OOM发生后，会生成堆快照来帮助排查问题。

文件如下：

![image-20231211154235938](https://pic-keboom.oss-cn-hangzhou.aliyuncs.com/img/image-20231211154235938.png)

### JProfiler

至于JProfiler的安装部署不赘述，只列几张图片看看大致监控的内容。

![image-20231211154448464](https://pic-keboom.oss-cn-hangzhou.aliyuncs.com/img/image-20231211154448464.png)

![image-20231211154503553](https://pic-keboom.oss-cn-hangzhou.aliyuncs.com/img/image-20231211154503553.png)

![image-20231211154523902](https://pic-keboom.oss-cn-hangzhou.aliyuncs.com/img/image-20231211154523902.png)

![image-20231211154539475](https://pic-keboom.oss-cn-hangzhou.aliyuncs.com/img/image-20231211154539475.png)

### jstat

jstat可实时查看堆状态。

先`jps -v`得到Java程序进程号，再`jstat -gcutil <pid>  <time interval>`（Example: jstat -gcutil 29218 3000 每隔三秒打印一次Java进程号为29218的gc信息）。

![image-20231211155704752](https://pic-keboom.oss-cn-hangzhou.aliyuncs.com/img/image-20231211155704752.png)

**S0，S1**：幸存者区

**E**：Eden区

**O**：Old 区

**M**：Metaspace

**CCS**：被编译的类所占元空间大小

**YGC**：Young GC 次数

**YGCT**：Young GC总时间

**FGC，FGCT**：Full GC次数，总时间

**GCT**：GC总时间

> 关于jstat -gc 和 jstat -gcutil 区别，主要是第一个显示实际大小，比如多少k。第二个显示百分比



### Arthas

使用[Arthas](https://arthas.aliyun.com/)也可以监控cpu，内存，gc等情况，具体可参考官方文档。也可参考我的这篇文章[关于使用Arthas排查问题](https://juejin.cn/post/7286750827896963087)

[关于docker中Java应用使用Arthas](https://bedecked-jumbo-3ec.notion.site/GPT-answer-7d331f77126d4c0581ff3c13f317497d?pvs=4)

---

无论使用什么方式获得JVM运行信息，最终我们要得到几组数据，用数据证明我们的调优确实有作用。



## 关于垃圾收集器

如果是JDK8，那么会有人说CMS是延时低的，Parallel GC等是吞吐量高的。但实际上还要经过测试才能确定。

对于JDK大于8的，比如JDK17等，可以看看G1、ZGC等收集器，测试其是否合适。

[GC progress from JDK 8 to JDK 17](https://kstefanj.github.io/2021/11/24/gc-progress-8-17.html)

## JVM OPTs 样例

-Duser.timezone=Asia/Shanghai 
-Xms6G -Xmx6G 
-XX:NewSize=3G -XX:MaxNewSize=3G 
-XX:SurvivorRatio=10
-XX:MetaspaceSize=2G -XX:MaxMetaspaceSize=2G
-XX:+HeapDumpOnOutOfMemoryError -XX:HeapDumpPath=./heapDump.hprof 
-XX:+PrintGCDetails -XX:+PrintGCTimeStamps -Xloggc:gc.log

Xms Xmx设置堆大小，两者一样可以避免扩容而导致一定延时
SurvivorRatio影响幸存者进入老年代的年龄阈值
MetaspaceSize设置一样可以防止扩容而导致延时
HeapDumpOnOutOfMemoryError OOM后输出堆快照
PrintGCDetails .... 打印GClog



## JVM调优案例

例子大部分来自于《深入理解Java虚拟机》，不说具体例子，只说造成结果

### 大对象直接进入老年代

导致老年代很快内存不够，导致频繁full GC，从而更多的延时

### 内存溢出

大量数据缓存到Java的堆中得不到释放，导致OOM。只要我们开启HeapDumpOnOutOfMemoryError 查看堆信息，基本上就能知道缓存了大量的什么Java对象。

### Direct Memory

我们一看到直接内存就能想到NIO，可以尝试扩大Direct Memory

### 外部命令导致资源占用

Java程序大量调用外部shell脚本

### socket 连接耗尽

发送的http请求，而响应却很慢才返回，导致socket耗尽

### 内存占用过大

数据结构问题，比如我们想查看某个人的一年的出勤率，我们可以看他未出勤的数据。比如我们就是要看一个人365天每一天的是否出勤，那么可以用map存365个key、value，但使用一个365长度的01字符串更节省空间。

### safepoint

文中说JVM对for循环有safepoint，对于for int 的是整个执行完才过safepoint，对于for long的是每一个循环就有safepoint。由于一个for int 执行时间过长导致 STW 过长。

详细可看：[HBase实战：记一次Safepoint导致长时间STW的踩坑之旅](https://juejin.cn/post/6844903878765314061)



---



## 总结

对于JVM调优，我们首先需要知道有什么样的问题，我们调优的目标是什么。一般有三个指标，吞吐量，延时，资源（footprint）。明确我们需要提高哪项指标后，才可进行相应的手段进行优化。

并且还有一个前提条件，那就是对于系统架构和代码层面的优化也做过了，对于数据库相关的优化也做过了，那么我们可以尝试调优JVM来优化相关指标。以为我们不能指望通过调优JVM来大幅提升性能。

仅仅从JVM角度说，如果我们要提高吞吐量，我们可以提高物理机性能，比如多开内存。或者换一个更注重吞吐量的垃圾收集器。当然也可以调节JVM参数来减少垃圾回收次数。

比如我们要减少延时，还是多开内存。或者换一个更注重降低延时的收集器。当然也是可以调节JVM参数减少垃圾回收次数等等。

如果我们要减少资源，如果可以忍受降低程序性能的话。那么我们能做的可能就是调节新生代，老年代比例等，比如我们的应用是朝生夕灭多（调大新生代），还是永久的对象更多（调大老年代）。



## Reference

[深入理解Java虚拟机：JVM高级特性与最佳实践（第3版）周志明.pdf]: youlink
[Guide to the Most Important JVM Parameters]: https://www.baeldung.com/jvm-parameters
[JVM Tuning: How to Prepare Your Environment for Performance Tuning]: https://sematext.com/blog/jvm-performance-tuning/
[从实际案例聊聊Java应用的GC优化]: https://tech.meituan.com/2017/12/29/jvm-optimize.html
[How to Properly Plan JVM Performance Tuning]: https://www.alibabacloud.com/blog/how-to-properly-plan-jvm-performance-tuning_594663
[Solving java.lang.OutOfMemoryError: Metaspace error]: https://www.mastertheboss.com/java/solving-java-lang-outofmemoryerror-metaspace-error/
[GC progress from JDK 8 to JDK 17]: https://kstefanj.github.io/2021/11/24/gc-progress-8-17.html
[HBase实战：记一次Safepoint导致长时间STW的踩坑之旅]: https://juejin.cn/post/6844903878765314061

