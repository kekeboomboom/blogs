## 对象（五种类型）

Redis的五种数据结构，字符串，list，hash，set，zset。其底层还有不同的编码来保证redis的灵活和效率。

![image-20210424121437765](C:\Users\kekeboomboom\AppData\Roaming\Typora\typora-user-images\image-20210424121437765.png)

![image-20210424121800890](C:\Users\kekeboomboom\AppData\Roaming\Typora\typora-user-images\image-20210424121800890.png)

## 数据库

Redis中默认分为16个数据库，客户端默认操作0号数据库。

## RDB

由于AOF更新频率更高，所以才优先使用AOF还原数据库状态。

![image-20210424134638843](C:\Users\kekeboomboom\AppData\Roaming\Typora\typora-user-images\image-20210424134638843.png)

设置自动间隔性保存![image-20210424135056256](C:\Users\kekeboomboom\AppData\Roaming\Typora\typora-user-images\image-20210424135056256.png)

RDB持久化通过保存数据库中的键值对来记录数据库状态，AOF持久化通过保存Redis服务器所执行的写命令来记录数据库状态。

![image-20210424141007661](C:\Users\kekeboomboom\AppData\Roaming\Typora\typora-user-images\image-20210424141007661.png)

## 客户端

![image-20210424144316346](C:\Users\kekeboomboom\AppData\Roaming\Typora\typora-user-images\image-20210424144316346.png)

## 服务端

![image-20210424144346832](C:\Users\kekeboomboom\AppData\Roaming\Typora\typora-user-images\image-20210424144346832.png)

![image-20210424144545146](C:\Users\kekeboomboom\AppData\Roaming\Typora\typora-user-images\image-20210424144545146.png)



## Sentinel

![image-20210424150627041](C:\Users\kekeboomboom\AppData\Roaming\Typora\typora-user-images\image-20210424150627041.png)

![image-20210424150702268](C:\Users\kekeboomboom\AppData\Roaming\Typora\typora-user-images\image-20210424150702268.png)

![image-20210424151443684](C:\Users\kekeboomboom\AppData\Roaming\Typora\typora-user-images\image-20210424151443684.png)