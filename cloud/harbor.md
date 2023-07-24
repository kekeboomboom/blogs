./prepare 是生成各种文件的脚本，harbor有很多服务，每个服务都有自己的配置文件或者数据文件，这个脚本会去生成。比如我们配置https时，key和crt等文件，他会那这个文件放到其他文件夹下，比如nginx的ssl配置目录。





harbor.yml 里面要配置 hostname  http port   和 https port、https 证书path

harbor整个体系里面有一个总的nginx 是对外的。docker-compose默认配置是对外监听80和443端口，然后转容器内8080,8443端口。

然后nginx在转给harbor其他服务。



after config，/etc/hosts add "ip  hostname"

![image-20230718144249787](https://pic-keboom.oss-cn-hangzhou.aliyuncs.com/img/image-20230718144249787.png)

![image-20230718144753236](https://pic-keboom.oss-cn-hangzhou.aliyuncs.com/img/image-20230718144753236.png)

so ,only update harbor.yml and /etc/hosts





