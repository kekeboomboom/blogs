# Ubuntu 离线安装软件包

关键词：apt-offline，Ubuntu，dpkg，.deb

本文使用的ubuntu20.04，当机器无法连接外网时，我们使用离线的方式安装软件包。

## 离线安装的软件包的几种方法

1. 下载.deb文件，然后dpkg 依次进行安装。这种方式需要我们注意依赖
2. apt-offline，这种方式不需要我们关注包的依赖，但是需要提前安装apt-offline这个工具
3. 一些图形化界面的软件，Ubuntu上的一些包管理图形化的软件

## apt-offline的方式

我不想关注.deb文件之间的依赖，因此我的方法是，先离线安装apt-offline，然后通过apt-offline去安装其他离线包。

### 离线安装apt-offline

[apt-offline deb文件下载](https://ubuntu.pkgs.org/20.04/ubuntu-universe-amd64/apt-offline_1.8.2-1_all.deb.html)

![image-20230911105124795](https://pic-keboom.oss-cn-hangzhou.aliyuncs.com/img/image-20230911105124795.png)

下载完成后，上传到offline的Linux服务器，然后使用dpkg命令安装：

![image-20230911105329521](https://pic-keboom.oss-cn-hangzhou.aliyuncs.com/img/image-20230911105329521.png)

这样我们就在offline的服务器上安装好了apt-offline工具

### 使用apt-offline离线安装其他软件包

[关于apt-offline的使用文档](https://docs.xubuntu.org/latest/user/C/offline-packages.html)

文档中讲的Updating Repositories，其实相当于 `apt update`

Installing a Package，相当于 `apt install`

Upgrading Your System，相当于 `apt upgrade`

结合其他网站上的回答：[askubuntu](https://askubuntu.com/questions/835655/install-a-program-with-apt-offline)