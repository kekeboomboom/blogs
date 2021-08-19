## 数据准备

测试数据的准备，使用MySQL官方示例测试数据。

[MySQL 官方示例测试数据导入](https://blog.csdn.net/active_it/article/details/80764821)

如果本机安装了mysql，那么直接找到文件夹下运行命令即可。

我用的是docker，所以需要将文件夹先复制到docker中，然后进入mysql容器，然后进入我们复制的文件夹下，执行命令

[Docker容器和本机之间的文件传输](https://blog.csdn.net/leafage_m/article/details/72082011)

## 验证索引的作用

我们对employees表进行操作，可以看到原始表中只有主键索引emp_no

![image-20210721205440661](https://gitee.com/keke518/MarkDownPicture/raw/master/img/image-20210721205440661.png)

我们查询同一条数据，一种我们不用索引，一种我们用主键索引，可以看到查询时间相差很大

![image-20210721205846544](https://gitee.com/keke518/MarkDownPicture/raw/master/img/image-20210721205846544.png)

![image-20210721205542172](https://gitee.com/keke518/MarkDownPicture/raw/master/img/image-20210721205542172.png)

![image-20210721205605800](https://gitee.com/keke518/MarkDownPicture/raw/master/img/image-20210721205605800.png)

通过explain来分析语句：

用emp_no去查询，可以看到我们用了主键索引![image-20210721205709320](https://gitee.com/keke518/MarkDownPicture/raw/master/img/image-20210721205709320.png)

用first_name去查询，我们啥也没用，type是ALL，表示全表扫描

![image-20210721205941327](https://gitee.com/keke518/MarkDownPicture/raw/master/img/image-20210721205941327.png)

那么我们创建索引：`ALTER TABLE employees ADD INDEX f_name(first_name, last_name)`

![image-20210721210145394](https://gitee.com/keke518/MarkDownPicture/raw/master/img/image-20210721210145394.png)

这次再通过firstname查，速度就快了：

![image-20210721210231363](https://gitee.com/keke518/MarkDownPicture/raw/master/img/image-20210721210231363.png)

用explain来分析：

![image-20210721210301239](https://gitee.com/keke518/MarkDownPicture/raw/master/img/image-20210721210301239.png)

他的意思说我们用了f_name 这个索引。

## 验证最左匹配

我们建立了f_name索引，现在我们分别执行下面两个语句：

![image-20210721211129182](https://gitee.com/keke518/MarkDownPicture/raw/master/img/image-20210721211129182.png)

![image-20210721211144839](https://gitee.com/keke518/MarkDownPicture/raw/master/img/image-20210721211144839.png)

![image-20210721211233136](https://gitee.com/keke518/MarkDownPicture/raw/master/img/image-20210721211233136.png)

以first_name为查询条件，可以看到走索引了

![image-20210721211323944](https://gitee.com/keke518/MarkDownPicture/raw/master/img/image-20210721211323944.png)

以last_name为查询条件，可以看到是全表搜索

## 验证不等于，不包含

![image-20210721212140704](https://gitee.com/keke518/MarkDownPicture/raw/master/img/image-20210721212140704.png)

![image-20210721212151195](https://gitee.com/keke518/MarkDownPicture/raw/master/img/image-20210721212151195.png)

看到走了全表扫描





未完待续~~~🐳🐳🐳🐳🐳🐳🐳🐳🐳🐳🐳🐳

