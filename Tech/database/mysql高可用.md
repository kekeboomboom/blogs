当前mysql高可用方案为双主+keepalived抢占式



相关资料：

https://www.cnblogs.com/lijiaman/p/13430668.html





keepalived抢占式：现在有mysqlA和mysqlB，A为主，B为辅。
当两个者都存活时，vip到A。
当A宕机后，服务器A的keepalived stop。vip route到B。
当A重新复活后，keepalived开启，由于是抢占式的，vip 会重新route 到A



A B节点的数据库互为主从，这样保证数据一致。