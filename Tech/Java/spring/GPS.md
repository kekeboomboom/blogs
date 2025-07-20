# GPS 方案

搜集网络上关于GPS的方案。

## redis + mysql 

redis 用来做设备或用户实时定位的查询。

mysql存储历史轨迹。存储时分两部分，一张表做实时查询用。一张表做备份用。如果需求为最多查询一个月的历史轨迹，那么实时查询表就只存储一个近一个月的轨迹数据。剩下的数据存到另一张表做备份。当然备份的方式有很多，你可以新建一张表存，或者csv存，或者存到其他数据库。

对于只查询历史轨迹的需求，那么在mysql中将经纬度存储成 decimal 类型即可。查询时根据时间戳进行查询（对时间戳建立索引，提高查询效率，当进行备份后，要重新构建索引）。

如果有类似查询两个点之间距离，当前点的方圆3公里内有哪些其他点位，查询球形曲面两点之间距离，存储一条路线，存一个区域，计算两个路线的交点等等几何上的计算。mysql支持的函数是相对较多的。



关于Geo数据类型[mysql spatial-types](https://dev.mysql.com/doc/refman/5.7/en/spatial-types.html)，各种数据库（mongodb，ES）支持的类型都一模一样，都是point，linestring，polygon，multi······。

mysql对于Geo优化：建立索引<u>**SPATIAL INDEX**</u>

关于Geo的相关的函数：[spatial-function](https://dev.mysql.com/doc/refman/5.7/en/spatial-function-reference.html)

插入一个point记录：`INSERT into geo_test(position) VALUES(ST_GeomFromText('POINT(39.984702 116.318417)'));`



## 时序数据库

关于时序数据库存储GPS轨迹数据，各种好处：

[知乎帖子](https://www.zhihu.com/question/24973158)

## mongodb

也支持Geo，数据类型与mysql一模一样，支持的函数相对于mysql较少

## ES

数据类型支持[ES Geo](https://www.elastic.co/guide/en/elasticsearch/reference/8.9/geo-shape.html)

数据分析[Geospatial analysis](https://www.elastic.co/guide/en/elasticsearch/reference/8.9/geospatial-analysis.html#geospatial-ingest)

[可视化](https://www.elastic.co/guide/en/elasticsearch/reference/8.9/geospatial-analysis.html#geospatial-visualize)，Kibana的支持，这是使用ES的一个亮点，如果想快速集成GPS信息的监控，可以选择ES。

下面是官方的可视化效果图：

![](https://www.elastic.co/guide/en/elasticsearch/reference/8.9/images/spatial/cumbre_vieja_eruption_dashboard.png)

ES的函数查询[ES Geo query](https://www.elastic.co/guide/en/elasticsearch/reference/8.9/geo-queries.html)