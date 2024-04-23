## 1.代码位置

### git

http://192.168.16.128/FusionCloud



前端项目：[FC-web](http://192.168.16.128/FusionCloud/FC-web)

后端项目：[fc-server](http://192.168.16.128/FusionCloud/fc-server)

### 服务器组件

![image-20240328140140601](C:\Users\keboo\AppData\Roaming\Typora\typora-user-images\image-20240328140140601.png)

Decklink、smh、NMS、ffmpeg、MultiServer等都是志强负责。

剩下的，仲健负责。



### 代码结构

我只说明后端Java代码结构。

![image-20240328140705923](C:\Users\keboo\AppData\Roaming\Typora\typora-user-images\image-20240328140705923.png)



#### controller层

![image-20240328140748574](C:\Users\keboo\AppData\Roaming\Typora\typora-user-images\image-20240328140748574.png)



#### service model层

![image-20240328140917094](C:\Users\keboo\AppData\Roaming\Typora\typora-user-images\image-20240328140917094.png)

service主要书写业务逻辑



![image-20240328140944095](C:\Users\keboo\AppData\Roaming\Typora\typora-user-images\image-20240328140944095.png)

mapper与数据库做交互



## 代码编译

需要下载idea，配置好Java环境，使用maven进行编译即可。



## 安装部署



### 第一次部署

放到211上 `\\192.168.16.211\package_release\GNGPublish\`



http://192.168.16.128/FusionCloud/fc-project 参照此项目的文档即可。以此进行安装。

安装Java 8环境 ：`apt install openjdk-8-jre-headless`



![image-20240329101754254](C:\Users\keboo\AppData\Roaming\Typora\typora-user-images\image-20240329101754254.png)

deployScript.tar.gz 解压运行cogentDeploy.sh 脚本





安装好程序之后，系统配置页面配置好ip和网口等

![image-20240329090134138](C:\Users\keboo\AppData\Roaming\Typora\typora-user-images\image-20240329090134138.png)

### 开机启动

在/etc/rc.local中，将两个启动脚本加进去。参考16.46

```
#!/bin/bash                                                                                           
                                                                                                      
/opt/java-server/java_start.sh >/dev/null 2>&1 &                                                      
sleep 1                                                                                               
/opt/java-server/start.sh >/dev/null 2>&1 &                                                           
                                                                                                      
exit 0
```

给rc.local可执行权限，java_start.sh和start.sh给可执行权限。



### 授权

smh 网关等程序授权。

### mediakit的一些参数说明

```
[ffmpeg]
bin = /opt/prog/decklink/bin/ffmpeg // 这个是用来截图，录像的ffmpeg需要注意初次安装时decklink的ffmpeg不能使用，需要ldd命令
cmd = sss%s -re -i %s -acodec aac -strict -2 -ar 48000 -ab 128 -c:v libx264 -f flv %s
log = ./ffmpeg/ffmpeg.log
restart_sec = 0
snap = %s -ss 00:00:05 -i %s -y -f mjpeg -vframes 1 %s

localIp 要配置成内网ip
externIP 要配置内网和外网ip，如果没有外网ip，则需只需要配置内网ip
```

> decklink ffmpeg 不能使用。进入 /etc/ld.so.conf  将 /opt/prog/decklink/libs 路径添加上去。执行 ldconfig 命令



## 升级

一般是前端页面修改，或者Java程序修改。

前端页面修改，自己 npm run build 将dist文件压缩后，替换服务器上 /opt/java-server/nginx 上html文件夹即可。



后端程序修改，修改后，idea进行maven

![image-20240328141628424](C:\Users\keboo\AppData\Roaming\Typora\typora-user-images\image-20240328141628424.png)

install 安装打包。

![image-20240328141811151](C:\Users\keboo\AppData\Roaming\Typora\typora-user-images\image-20240328141811151.png)

install后会生成target目录，lib 和 cogent-admin.jar 是我们需要的。将lib目录和 cogent-admin.jar 将服务器 /opt/java-server/cogent-admin 目录的lib 和 cogent-admin.jar 替换就行。



替换后重启java程序， 执行/opt/java-server/cogent-admin/ry.sh restart



## 主要框架

springboot mybatis-plus mysql redis ZLmediakit minio 

https://github.com/ZLMediaKit/ZLMediaKit/wiki

https://min.io/docs/minio/linux/index.html

https://spring.io/projects/spring-boot

https://baomidou.com/

## 程序设计

![image-20240328140140601](C:\Users\keboo\AppData\Roaming\Typora\typora-user-images\image-20240328140140601.png)



vue，前端组件

nginx，反向代理

fc-server，Java服务，会调用网关，smh，nms等接口，做一些操作

minio，对象存储，做文件上传

ftp-server，做断点续传

mobicaster，手机app接入

mediakit，流媒体服务器

mysql、redis，数据库

ffmpeg，做直播预览截图



## 数据库表单

![image-20240328160318564](C:\Users\keboo\AppData\Roaming\Typora\typora-user-images\image-20240328160318564.png)

backpack，背包相关数据

backpack_blacklist，背包黑名单

call_group，群组通话

call_group_member，群组成员

dest_device，路由目的地

foldback_source，返送源

gps_info，gps数据

live_snap，直播截图

media_current_*，设备状态，配置信息

minio_*，相关文件上传等

route_*，smh，路由相关



字段说明，每个表的每个字段都有注释。
