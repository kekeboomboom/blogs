## MySQL日志

[MySQL日志系统](https://blog.csdn.net/u010002184/article/details/88526708)

redo_log，undo_log，bin_log？

[redo_log崩溃恢复](https://cloud.tencent.com/developer/article/1417482#:~:text=%E5%B4%A9%E6%BA%83%E6%81%A2%E5%A4%8D.,%E5%B4%A9%E6%BA%83%E6%81%A2%E5%A4%8D%E8%83%BD%E5%8A%9B%E6%98%AF%E6%8C%87InnoDB%E5%8F%AF%E4%BB%A5%E4%BF%9D%E8%AF%81%E6%95%B0%E6%8D%AE%E5%BA%93%E5%9C%A8%E5%BC%82%E5%B8%B8%E5%B4%A9%E6%BA%83%E9%87%8D%E5%90%AF%E5%90%8E%E7%9A%84%E7%8A%B6%E6%80%81%E5%92%8C%E4%BD%BF%E7%94%A8binlog%E6%96%87%E4%BB%B6%E6%81%A2%E5%A4%8D%E5%87%BA%E6%9D%A5%E7%9A%84%E6%95%B0%E6%8D%AE%E5%BA%93%E7%8A%B6%E6%80%81%E4%BF%9D%E6%8C%81%E4%B8%80%E8%87%B4%E3%80%82.)

[binlog redolog undolog](https://mp.weixin.qq.com/s/Lx4TNPLQzYaknR7D3gmOmQ)

## MySQL查询

[菜鸟教程SQL内连接](https://www.runoob.com/sql/sql-join-inner.html)

[exist和in区别](https://blog.csdn.net/qq_27409289/article/details/85963089)

[sql语句优化](https://zhuanlan.zhihu.com/p/265852739)

## MySQL索引

[覆盖索引](https://juejin.cn/post/6844903967365791752)

 索引类型：

- 主键索引（innodb的主键为聚集索引，所有数据都存放在b+树叶节点）
- 二级索引（辅助索引，叶节点data域记录主键的值，然后根据主键值，在主索引查找，也就是回表）：唯一索引，普通索引，全文索引，联合索引

[索引优化](https://zhuanlan.zhihu.com/p/61687047)

[索引下推](https://www.cnblogs.com/Chenjiabing/p/12600926.html)

索引下推：对于表id，name，age。有索引index（name，age）

查询：select * from name like “王%” and age = 10

如果不用索引下推，那么在存储引擎查询到 name like “王%” ，拿到数据到server层，然后再以age=10为条件，再去存储引擎查询。那比如我们查询到3000个姓王的，然后再从这3000个找到age=10的，比如有13个人。

如果用索引下推，那么在存储引擎查询到name like “王%” and age = 10的这13个人主键id，然后再去主键索引查询数据就好了。



### 对于%*%的优化

[男朋友连模糊匹配like %%怎么优化都不知道 (qq.com)](https://mp.weixin.qq.com/s/ygvuP35B_sJAlBHuuEJhfg)

一种时通过建立全文索引，select ------ match -----

第二种通过reverse建立虚拟列，然后正这查，反转后查就行了。



## MySQL主备

[MySQL主备](https://blog.csdn.net/qq_40378034/article/details/91125768)

[主从复制，读写分离](https://zhuanlan.zhihu.com/p/199217698)

MySQL的主从复制？

![image-20210910140521176](https://gitee.com/keke518/MarkDownPicture/raw/master/img/20210910140528.png)

## MySQL锁

[MySQL中的锁（表锁、行锁](https://www.cnblogs.com/chenqionghe/p/4845693.html))

[MySQL锁总结](https://zhuanlan.zhihu.com/p/29150809)

innodb的意向锁，为了允许行锁和表锁共存，实现多粒度锁机制，有了意向锁，意向锁都是表锁

- 意向共享锁（IS）：事务打算给数据行加行共享锁，事务在给一个数据行加共享锁前必须先取得该表的 IS 锁。
- 意向排他锁（IX）：事务打算给数据行加行排他锁，事务在给一个数据行加排他锁前必须先取得该表的 IX 锁。

Next-Key Locking ：是当前行锁和间隙锁的结合



## MySQL事务

[可重复读能够防止幻读](https://cloud.tencent.com/developer/article/1506516)

[mvcc](https://www.jianshu.com/p/d67f0329d3bf)

[MySQL事务的ACID](https://www.cnblogs.com/kismetv/p/10331633.html)

[MySQL的可重复读级别能解决幻读吗 - 宁愿呢 - 博客园 (cnblogs.com)](https://www.cnblogs.com/liyus/p/10556563.html)

事务的四种隔离级别？脏读，不可重复读，幻读是什么？

mvcc原理？

MySQL的ACID如何实现的？

MySQL默认隔离级别？在默认隔离级别下能否解决幻读？

### MVCC，undo，快照读，当前读

mvcc使用的快照存储在undo log链中，在select时会生成进行快照读，通过readview读取合适的版本数据。

在mvcc中select都是快照读，不需要加锁。

mvcc其他会对数据库进行修改的操作insert，update，delete等加锁操作，从而读取最新数据，也就是当前读。

**在 InnoDB 存储引擎中，SELECT 操作的不可重复读问题通过 MVCC 得到了解决，而 UPDATE、DELETE 的不可重复读问题通过 Record Lock 解决，INSERT 的不可重复读问题是通过 Next-Key Lock（Record Lock + Gap Lock）解决的。**

Record Lock：锁定一个记录上的索引，而不是记录本身

Gap Lock：锁定索引之间的间隙，但是不包含索引本身。

Next-Key Lock：它是 Record Locks 和 Gap Locks 的结合。

## MySQL架构

[MySQL架构](https://www.cnblogs.com/michael9/p/12497992.html)



## MySQL存储引擎

### innodb

- 支持事务，四种隔离级别
  - 读未提交，不需要加任何锁
  - 串行化，读写锁都加
  - RC和RR ，通过MVCC支持高并发，通过 MVCC + Next-Key Locking 防止幻读。

- 默认行级锁，行级锁粒度更小，并发更高
- 主键索引为聚簇索引（在索引中保存数据，而对于mysiam的主键是根据id查到地址，根据地址去拿到数据）



### b+树

innodb的数据结构就是b+树。

- 每个内部节点（非叶子节点）只存储索引键，叶子节点存储数据。
  - 这样每次磁盘io就能读取更多的键，一次io查询范围就更广，io次数更少，效率更高
  - 只有叶节点才存储数据，那么每次查询必须从根到叶子节点才能获得数据，因此查询更稳定
- 叶节点之间有指针相连，方便范围查询。



AVL树和红黑树都是二叉树，树高太高，io效率太低。

AVL必须保持左右子树高度差不超过1，因此插入删除要通过旋转保持平衡，旋转很耗时，因此适合插入删除少，查找多的场景。

红黑树从根节点到叶节点最长路径不超过最短路径的2倍，相对于AVL树不要求那么平衡，因此旋转次数较少，适合查找少，插入删除多的场景。





## 三范式

第一范式：要求每一列都是一个字段

第二范式：对于键码不能有部份依赖。比如一张表有学生id，name，age，sex，课程id，课程name，

那么键码就是（学生id，课程id），name只部分依赖于学生id，课程name部分依赖于课程id，因此需要拆分为学生表和课程表

第三范式：对于键码不能有传递依赖，比如表有学生id，name，age，班级，班主任，那么有传递依赖：学生id-->班级-->班主任，因此拆分学生表和班级表





## SQL优化

内联子查询

```
select id,(select rule_name from member_rule limit 1) as rule_name, member_id, member_type, member_name, status  from member_info m where status = 1 and create_time between '2020-09-02 10:00:00' and '2020-10-01 10:00:00';
```

索引列运算

```
select account_no, balance from accounts where balance + 100 = 10000 and status = 1;
```

类型转换

```
#user_id是bigint类型，传入varchar值发生了隐式类型转换，可以走索引。
select id, name , phone, address, device_no from users where user_id = '23126';
#card_no是varchar(20)，传入int值是无法走索引
select id, name , phone, address, device_no from users where card_no = 2312612121;
```

函数运算

```
select DATE_FORMAT(create_time, '%Y-%m-%d'), count(*) from users where create_time between '2020-09-01 00:00:00' and '2020-09-30 23:59:59' group by DATE_FORMAT(create_time, '%Y-%m-%d');
```

复合索引，最左匹配，MySQL遵循的是索引最左匹配原则，对于复合索引，从左到右依次扫描索引列，到遇到第一个范围查询（>=, >,<, <=, between ….. and ….）就停止扫描，

