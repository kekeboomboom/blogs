## SDS

作为字符串的一种实现。类比Java中HashMap作为一种数据结构，其内部的键值对也是需要类型的，比如String类型，HashMap<String, String>如此形式。

SDS作为一种专门设计的不同于c语言的字符串，有空间预分配和惰性空间释放功能，其目的都是为了尽量少去改动字节数组，以空间换时间。

## 链表

我们只从Redis数据类型说，链表应用于列表，对于一个列表长度很长或者value存储的字符串很长时会使用链表。

### 链表list和链表节点listnode

listnode：前驱节点，后继节点，value。典型的双向列表

list：头节点指针，尾节点指针，链表长度，三个函数



## 字典

应用于数据类型hash（就是用来存储对象很合适的）和String类型。

字典：字典dict，哈希表dictht，哈希表节点dictEntry

dict：type，privdata暂时不知道有何用，ht存储哈希表，rehashidx重新hash时使用

dictht：table数组指针（指向一个dictEntry结构），table数组大小，used已有节点数量

dictEntry：键key，值union，next指针（指向下一个dictEntry，类似拉链法解决hash冲突）

