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





---

