

## Redis是什么？

[Redis是什么](https://mp.weixin.qq.com/s/EjDeypra_d9Tfsn-WkJZdw#:~:text=Redis%E9%87%87%E7%94%A8%E7%9A%84%E6%98%AF%E5%9F%BA%E4%BA%8E%E5%86%85%E5%AD%98%E7%9A%84%E9%87%87%E7%94%A8%E7%9A%84%E6%98%AF%E5%8D%95%E8%BF%9B%E7%A8%8B%E5%8D%95%E7%BA%BF%E7%A8%8B%E6%A8%A1%E5%9E%8B%E7%9A%84%20KV%20%E6%95%B0%E6%8D%AE%E5%BA%93%EF%BC%8C%E7%94%B1C%E8%AF%AD%E8%A8%80%E7%BC%96%E5%86%99%EF%BC%8C%E5%AE%98%E6%96%B9%E6%8F%90%E4%BE%9B%E7%9A%84%E6%95%B0%E6%8D%AE%E6%98%AF%E5%8F%AF%E4%BB%A5%E8%BE%BE%E5%88%B0100000%2B%E7%9A%84QPS%EF%BC%88%E6%AF%8F%E7%A7%92%E5%86%85%E6%9F%A5%E8%AF%A2%E6%AC%A1%E6%95%B0%EF%BC%89%E3%80%82)

## 对象

### 字符串

自增，键值对。

SDS数据结构记录长度，已经使用，和总共长度，并且提前多余出容量，防止一直扩容缩容。

字符串对象key，value整体是字典。key和value的都是字符串结构为SDS。

使用场景：缓存，计数器，session



### list

底层就是双向链表

使用场景：粉丝列表，评论列表，lrange命令支持分页查询，消息队列

### hash

存储对象。

hash表结构，拉链法解决冲突。



字典结构和rehash过程都在了：dict里面有两个ht，第一个ht是平时存放数据的，第二个ht是rehash时使用的。每个dictht种有一个table数组，数组种每个元素都指向一个dictEntry，如果发成冲突，通过拉链法dictEntry通过next指针形成链表。

**rehash过程：**

- 当发生rehash，为第二个ht分配空间，
- 然后计算hash值和索引值，将键值对放到第二个ht中。
- 所有数据迁移完成之后，释放ht[0]，将ht[1]设置为ht[0]，并再ht[1]创建空白哈希表。

![image-20210911090642584](https://gitee.com/keke518/MarkDownPicture/raw/master/img/20210911090642.png)

![image-20210911090735297](https://gitee.com/keke518/MarkDownPicture/raw/master/img/20210911090950.png)







### set

使用场景：共同关注，共同好友等。





### zset

使用场景：排行榜，带权重的消息队列

[Redis—跳跃表 (qq.com)](https://mp.weixin.qq.com/s/NOsXdrMrWwq4NTm180a6vw)

#### 用跳表而不是红黑树？

1. 性能：对于树形结构，插入删除就要考虑reblance，相对于跳表变化只涉及局部
2. 实现：跳表实现更简单，更直观

多层链表，比如每相邻两个节点增加一个指针，每相邻四个节点增加一个指针，这样就有了多层链表，我们查询时，先查最上层的，这样一层层缩小范围，效率就更高了。

#### 跳跃表

跳跃表：如果我们严格遵守上下层2：1的关系，那么我们插入的时候就需要调整结构。为了避免这个问题，我们不要求严格遵守上下的这个关系。为每个节点随机出一个层数，每次插入时只需要修改前后的节点，这样降低插入复杂度。

![](https://mmbiz.qpic.cn/mmbiz_png/ia1kbU3RS1H5ZLiaicqeR9mzkQuQLwvtFfQ5qUqf8c0vC3bfbc710Tz6iadcOlDYb39pApOUP9pCaUDQtuicUn9Jibvg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

redis跳跃表实现：

```c
/* ZSETs use a specialized version of Skiplists */
typedefstruct zskiplistNode {
    // value
    sds ele;
    // 分值
    double score;
    // 后退指针
    struct zskiplistNode *backward;
    // 层
    struct zskiplistLevel {
        // 前进指针
        struct zskiplistNode *forward;
        // 跨度
        unsignedlong span;
    } level[];
} zskiplistNode;

typedefstruct zskiplist {
    // 跳跃表头指针
    struct zskiplistNode *header, *tail;
    // 表中节点的数量
    unsignedlong length;
    // 表中层数最大的节点的层数
    int level;
} zskiplist;
```

以score排序，节点存储的值ele。

![image-20210911094433069](https://gitee.com/keke518/MarkDownPicture/raw/master/img/20210911094433.png)





Redis的五种数据结构，字符串，list，hash，set，zset。其底层还有不同的编码来保证redis的灵活和效率。

![image-20210424121437765](C:\Users\kekeboomboom\AppData\Roaming\Typora\typora-user-images\image-20210424121437765.png)

![image-20210424121800890](C:\Users\kekeboomboom\AppData\Roaming\Typora\typora-user-images\image-20210424121800890.png)





### 布隆过滤器

[JavaFamily/布隆过滤器(BloomFilter).md at master · AobingJava/JavaFamily (github.com)](https://github.com/AobingJava/JavaFamily/blob/master/docs/redis/布隆过滤器(BloomFilter).md)

原理：

缺点：存在误判，删除困难

应用：缓存穿透，爬虫过滤已经抓的url，垃圾邮件过滤。

### HyperLoglog

基数统计。比如我们要统计今天有多少用户访问了我们的网站。对于相同的ip地址属于同一个用户，那么我们想要统计有多少种不同的ip地址，这就可以用到基数统计了。

优点：大数据下更节省内存，当然有一定误差。对于基数统计有多种算法，HyperLoglog只是其中一种。



## 多个系统同时操作redis如何保证顺序？

要保证分布式系统操作顺序，那就分布式锁喽，比如用zookeeper分布式锁。



## redis线程模型

**Redis** 内部使用文件事件处理器 `file event handler`，这个文件事件处理器是单线程的，所以 **Redis** 才叫做单线程的模型。它采用 IO 多路复用机制同时监听多个 **Socket**，根据 **Socket** 上的事件来选择对应的事件处理器进行处理。

文件事件处理器的结构包含 4 个部分：

- 多个 **Socket**
- IO 多路复用程序
- 文件事件分派器
- 事件处理器（连接应答处理器、命令请求处理器、命令回复处理器）

多个 **Socket** 可能会并发产生不同的操作，每个操作对应不同的文件事件，但是 IO 多路复用程序会监听多个 **Socket**，会将 **Socket** 产生的事件放入队列中排队，事件分派器每次从队列中取出一个事件，把该事件交给对应的事件处理器进行处理。

### redis为什么单线程？

**Redis中只有网络请求模块和数据操作模块是单线程的。而其他的如持久化存储模块、集群支撑模块等是多线程的。**

redis的性能瓶颈在于IO，那么要提升IO并不是必须要引入多线程，多线程的弊端：

- 引入多线程就要考虑共享资源并发问题，带来了复杂性，开发，维护成本更高
- 单线程不需要考虑线程切换的性能开销
- IO多路复用也能提升IO效率

### redis6.0引入多线程

虽然已经有很好的性能，但是我们还要求他更好。而经过分析，限制Redis的性能的主要瓶颈出现在网络IO的处理上。虽然有多路IO模型，但是**多路复用的IO模型本质上仍然是同步阻塞型IO模型**

因此采用多个IO线程来处理网络请求，提升网络请求处理的并行度，进而提升整体性能。

由于读写命令仍然是单线程的，因此考虑并发带来的线程安全问题。

### redis为什么快？

- 1、完全基于内存，绝大部分请求是纯粹的内存操作，非常快速。
- 2、数据结构简单，对数据操作也简单，如哈希表、跳表都有很高的性能。
- 3、采用单线程，避免了不必要的上下文切换和竞争条件，也不存在多进程或者多线程导致的切换而消耗 CPU
- 4、使用多路I/O复用模型

## 数据库

Redis中默认分为16个数据库，客户端默认操作0号数据库。

## 持久化

RDB：对数据进行周期性的持久化，比如5分钟一次，记录数据库中的数据

- 优点
  - 周期性的持久化，可以做冷备份
  - 同步数据时fork子进程，对性能影响小
- 缺点
  - 五分钟一次，那么再上次同步到现在的五分钟内数据可能低丢掉

AOF：对每条写入命令为日志，以追加的方式持久化到磁盘，比如每1秒钟持久化一次

- 优点
  - 每秒一次生成快照，最多丢失一秒的数据
  - 以`append-only`的方式去写数据，写入性能好。
- 缺点
  - 一样的数据，**AOF**文件比**RDB**还要大。

两者都开启时，使用AOF，因为AOF数据更完整

真当出现问题时，使用RDB先恢复，然后使用AOF做补全数据。











## 客户端

![image-20210424144316346](C:\Users\kekeboomboom\AppData\Roaming\Typora\typora-user-images\image-20210424144316346.png)

## 服务端

![image-20210424144346832](C:\Users\kekeboomboom\AppData\Roaming\Typora\typora-user-images\image-20210424144346832.png)

![image-20210424144545146](C:\Users\kekeboomboom\AppData\Roaming\Typora\typora-user-images\image-20210424144545146.png)



## 分布式高可用

### Sentinel

第一个高可用：可以启动多个master node，每个master node可以有多个salve node

第二个就是sentinel：

![image-20210424150627041](C:\Users\kekeboomboom\AppData\Roaming\Typora\typora-user-images\image-20210424150627041.png)

![image-20210424150702268](C:\Users\kekeboomboom\AppData\Roaming\Typora\typora-user-images\image-20210424150702268.png)

![image-20210424151443684](C:\Users\kekeboomboom\AppData\Roaming\Typora\typora-user-images\image-20210424151443684.png)

sentinel功能：

- 集群监控：负责监控master和slave是否正常工作
- 故障转移：如果某个master挂掉了，则会自动选出一个slave来作为主节点
- 配置中心：如果发生故障转移，则通知client更新master地址



### 主从数据如何同步？

你启动一台slave 的时候，他会发送一个**psync**命令给master ，如果是这个slave第一次连接到master，他会触发一个全量复制。master就会启动一个线程，生成**RDB**快照，还会把新的写请求都缓存在内存中，**RDB**文件生成后，master会将这个**RDB**发送给slave的，slave拿到之后做的第一件事情就是写进本地的磁盘，然后加载进内存，然后master会把内存里面缓存的那些新命名都发给slave。

### 如果数据传输过程断网了？或数据库挂了如何？

传输过程中有什么网络问题啥的，会自动重连的，并且连接之后会把缺少的数据补上的。

**大家需要记得的就是，RDB快照的数据生成的时候，缓存区也必须同时开始接受新请求，不然你旧的数据过去了，你在同步期间的增量数据咋办？是吧？**





## redis的过期key删除策略和内存淘汰机制

### 过期key删除策略：

- 定期删除：默认100ms随机抽取设置了过期时间的key，检查是否过期
- 惰性删除：等到来查询的时候，看看是不是过期了，过期了那就删除。

### 那么如果定期也没删，我也没查询，总之内存满了，怎么办？

内存淘汰机制：

- **noeviction**:返回错误当内存限制达到并且客户端尝试执行会让更多内存被使用的命令（大部分的写入指令，但DEL和几个例外）

- **allkeys-lru**: 尝试回收最少使用的键（LRU），使得新添加的数据有空间存放。

- **volatile-lru**: 尝试回收最少使用的键（LRU），但仅限于在过期集合的键,使得新添加的数据有空间存放。

- **allkeys-random**: 回收随机的键使得新添加的数据有空间存放。

- **volatile-random**: 回收随机的键使得新添加的数据有空间存放，但仅限于在过期集合的键。

- **volatile-ttl**: 回收在过期集合的键，并且优先回收存活时间（TTL）较短的键,使得新添加的数据有空间存放。

  如果没有键满足回收的前提条件的话，策略**volatile-lru**, **volatile-random**以及**volatile-ttl**就和noeviction 差不多了。



## redis怎么保证和数据库的数据一致

**Cache Aside Pattern**

- 读的时候，先读缓存，缓存没有的话，就读数据库，然后取出数据后放入缓存，同时返回响应。
- 更新的时候，**先更新数据库，然后再删除缓存**。

### 为什么是删除而不是更新缓存？

如果更新操作简单，仅仅是将值修改为另一个值，那么更新和删除差不多。但是某些更新操作复杂，更新的值需要进行计算，更新一个缓存还涉及数据库中其他数据，此时更新cache消耗就大了。

所以直接删除，等到查询时再更新。



### 对于读写我们没什么问题，讨论更新

单节点下讨论：

[如何保证缓存(redis)与数据库(MySQL)的一致性-阿里云开发者社区 (aliyun.com)](https://developer.aliyun.com/article/712285)

> **【单节点下两种方案对比】**
> **先淘汰cache，再更新数据库：**
>   采用同步更新缓存的策略，可能会导致数据长时间不一致，如果用延迟双删来优化，还需要考虑究竟需要延时多长时间的问题——读的效率较高，但数据的一致性需要靠其它手段来保证
>   采用异步更新缓存的策略，不会导致数据不一致，但在数据库更新完成之前，都需要到数据库层面去读取数据，读的效率不太好——**保证了数据的一致性，适用于对一致性要求高的业务**
> **先更新数据库，再淘汰cache：**
>   无论是同步/异步更新缓存，都不会导致数据的最终不一致，在更新数据库期间，cache中的旧数据会被读取，可能会有一段时间的数据不一致，但读的效率很好——**保证了数据读取的效率，如果业务对一致性要求不是很高，这种方案最合适**



## redis分布式锁

分布式锁的实现方式：

- MySQL中的悲观锁
- Zookeeper有序节点
- Redis单线程执行命令，命令的执行为串行

比如减库存这个操作，对于单机多线程，只需要sychronized和lock解决。

单机多线程，我们先考虑redis单机情况，使用setnx命令，第一个拿到锁的就去执行，同时为了防止执行过程中挂掉了，要对锁设置过期时间。加锁解锁时设置uuid，直到是谁加锁谁解锁。解锁使用lua。

### 锁超时

对锁不设置超时时间，那么获得锁的服务A挂掉了，那么服务B永远获取不到锁。

那么设置超时时间，有了超时时间问题又来了，服务A获得了锁并且执行业务逻辑，但是业务时间过长，导致锁过期自动释放了，服务B也能获得锁。导致服务A服务B都能执行临界区代码。一个解决办法是尽量业务时间不要太长，那我业务时间是不知道的，我不能每次使用分布式锁的时候还要计算以下业务时间以此来设置超时时间吧，那么redisson实现一个方案，**加锁时，先设置一个过期时间，然后我们开启一个「守护线程」，定时去检测这个锁的失效时间，如果锁快要过期了，操作共享资源还未完成，那么就自动对锁进行「续期」，重新设置过期时间。**Redisson 是一个 Java 语言实现的 Redis SDK 客户端，在使用分布式锁时，它就采用了「自动续期」的方案来避免锁过期，这个守护线程我们一般也把它叫做「看门狗」线程。

第二点，业务时间不长，但是由于JVM进行了GC，导致服务A时间超时：

![](https://mmbiz.qpic.cn/mmbiz_png/ia1kbU3RS1H6lP6XUgnFmTwicdR7YlhJsfIm71stkfNGwaBUgp8qHxygqicCfow4FBANMnAEibSibOYdBEzQfPZzsicQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

### 单点/多点问题

刚才都是在redis单点基础上讨论，为了保证可用性，提出了RedLock算法：

[Redlock（redis分布式锁）原理分析 - 云+社区 - 腾讯云 (tencent.com)](https://cloud.tencent.com/developer/article/1431873)

假设有5个完全独立的redis主服务器

1.获取当前时间戳

2.client尝试按照顺序使用相同的key,value获取所有redis服务的锁，在获取锁的过程中的获取时间比锁过期时间短很多，这是为了不要过长时间等待已经关闭的redis服务。并且试着获取下一个redis实例。

   比如：TTL为5s,设置获取锁最多用1s，所以如果一秒内无法获取锁，就放弃获取这个锁，从而尝试获取下个锁

3.client通过获取所有能获取的锁后的时间减去第一步的时间，这个时间差要小于TTL时间并且至少有3个redis实例成功获取锁，才算真正的获取锁成功

4如果成功获取锁，则锁的真正有效时间是 TTL减去第三步的时间差 的时间；比如：TTL 是5s,获取所有锁用了2s,则真正锁有效时间为3s(其实应该再减去时钟漂移);

5.如果客户端由于某些原因获取锁失败，便会开始解锁所有redis实例；因为可能已经获取了小于3个锁，必须释放，否则影响其他client获取锁

锁释放时，要解锁所有redis实例，因为在分布式系统中，网络问题导致即使加锁成功了，但是看到的消息却是加锁失败。



### 那对于RedLock的分布式实现就一定安全么？

答案当然时不安全，对于分布式问题总会遇到如下问题：

- N：Network Delay，网络延迟
- P：Process Pause，进程暂停（GC）
- C：Clock Drift，时钟漂移

比如网络延迟会造成，已经在master节点设置了锁，但是client却以为没有获得该master的锁。

比如GC，导致服务A和服务B都同时进入了临界区

比如时钟飘逸，我们对于TTL，对于是否成功获得锁，是否超时等，都需要对时间进行判断。



## redis常见线上故障及解决方案

### 缓存雪崩

- 是什么：缓存在同一时间大面积失效，导致大量请求打到数据库上
- 解决方案：对每个key的失效时间设置随机值，对于热点数据设置永不失效

### 缓存穿透

- 是什么：大量请求的key不存在，导致大量请求到数据库
- 解决方案：设置参数校验，将查询不到的key设置为缓存，**布隆过滤器**

### 缓存击穿

- 是什么：对于一个热点key，在他失效的瞬间，导致大量请求穿过缓存到了数据库
- 解决方案：设置热点数据永远不过期，或者加上互斥锁就能搞定了





## redis为什么变慢？

[Redis为什么变慢了？一文讲透如何排查Redis性能问题 | 万字长文 (qq.com)](https://mp.weixin.qq.com/s/rw42cFbJXwPtsGiqkFErfw)

排除网络因素，假设原因就在redis上。

- 可以对redis做基准测试，看是否真的是redis慢
- 如果真的是redis慢，使用了复杂度高的命令？通过查询慢日志，查看那些命令执行的慢，比如sort，suion等聚合函数
- 操作bigkey，就是存储的value太大了，`$ redis-cli -h 127.0.0.1 -p 6379 --bigkeys -i 0.01`
- 集中过期key，在一时间大量key过期，redis会先进行删除key（这个操作耗时，但是他又不是命令，所以不会记录在慢日志中），然后再执行命令
  - 那遇到这种情况，如何分析和排查？此时，你需要检查你的业务代码，是否存在集中过期 key 的逻辑。一般集中过期使用的是 expireat / pexpireat 命令，你需要在代码中搜索这个关键字。



## redis实现消息队列？

三种方法：

- list
- stream
- pub/sub

## redis在我的项目中的应用

文章点赞和关注，通过set和zset实现。个人获得的点赞数通过string类型记录。

日志记录，想要直到当前用户信息，每次查数据库效率太低，将用户信息缓存在redis。如用户名等

邮箱验证，无论是登录还是注册，将邮箱验证码放到redis设置过期时间。





