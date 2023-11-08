# minio 支持object搜索方案

minio支持上传时对object打标签，查询时可以根据标签做筛选。但是有ftp上传文件的需求，导致无法给object打标签。并且也不清楚minio对于根据标签的筛选性能如何，因此我们打算将object的对象的数据放到数据库。在数据库中对object进行筛选。

## docker部署

```
mkdir -p ~/minio/data

docker run \
   -d \
   --network=host \
   --name minio \
   -v /opt/minio/data:/data \
   -e "MINIO_ROOT_USER=ROOTNAME" \
   -e "MINIO_ROOT_PASSWORD=CHANGEME123" \
   quay.io/minio/minio server /data --console-address ":9090" \
   --address=":9002" \
   --ftp="address=:8021" \
   --ftp="passive-port-range=30000-40000"
```

9090端口是minio console网页的端口，登录此网页用户名密码就是我们设置的ROOTNAME  CHANGEME123

9002是我们程序调用minio sdk的端口，根据自己机器的情况按需设置。

ftp参数表示我们开启ftp，并使用8021最为ftp服务器端口。

登录9090网页后，申请Access key，调用minio sdk时需要secret key、Access key等。



## Bucket Notification

我们通过订阅minio的object events来做数据同步。

[bucket notification](https://min.io/docs/minio/linux/administration/monitoring/bucket-notifications.html)

minio提供了多种方式，我们最终使用的webhook方式，但是我们暂时先说一下mysql方式：

```
docker run \
   -d \
   --network=host \
   --name minio \
   -v /opt/minio/data:/data \
   -e "MINIO_ROOT_USER=ROOTNAME" \
   -e "MINIO_ROOT_PASSWORD=CHANGEME123" \
   -e MINIO_NOTIFY_MYSQL_ENABLE_PRIMARY="on" \
   -e MINIO_NOTIFY_MYSQL_DSN_STRING_PRIMARY="root:root@tcp(192.168.16.46:3306)/cogent-admin" \
   -e MINIO_NOTIFY_MYSQL_TABLE_PRIMARY="minio_events" \
   -e MINIO_NOTIFY_MYSQL_FORMAT_PRIMARY="namespace" \
   quay.io/minio/minio server /data --console-address ":9090" \
   --address=":9002" \
   --ftp="address=:8021" \
   --ftp="passive-port-range=30000-40000"
```

MINIO_NOTIFY_MYSQL_DSN_STRING_PRIMARY 配置我们的mysql数据库，然后进入docker容器

```
docker exec -it minio bash
chmod 777 /opt/bin/mc  
mc alias set myminio http://192.168.16.46:9002 ROOTNAME CHANGEME123
mc admin info --json myminio

mc event add myminio/cogent arn:minio:sqs::PRIMARY:mysql \
  --event put

mc event add myminio/cogent arn:minio:sqs::PRIMARY:mysql \
  --event delete

mc event ls myminio/cogent arn:minio:sqs::PRIMARY:mysql
```

> 注意事项：我们必须先至少创建一个bucket才能mc event add。进入容器后，我们要给/opt/bin/mc 执行权限。mc event add 的配置好像会持久化，如果重新启动一个容器，event仍然在，这时就不用在执行mc event add命令了，当然我们可以mc event ls去确定一下evnet是否添加成功

最后我们的结果，minio发送给mysql中的数据：

![image-20230908110237965](https://pic-keboom.oss-cn-hangzhou.aliyuncs.com/img/image-20230908110237965.png)

value：

```json
{
    "Records": [
        {
            "s3": {
                "bucket": {
                    "arn": "arn:aws:s3:::cogent",
                    "name": "cogent",
                    "ownerIdentity": {
                        "principalId": "ROOTNAME"
                    }
                },
                "object": {
                    "key": "jetbra.zip",
                    "eTag": "8943434aec7868e6e16d36209ca47fab",
                    "size": 172138,
                    "sequencer": "177F6C96E9864D53",
                    "contentType": "application/x-zip-compressed",
                    "userMetadata": {
                        "content-type": "application/x-zip-compressed"
                    }
                },
                "configurationId": "Config",
                "s3SchemaVersion": "1.0"
            },
            "source": {
                "host": "192.168.16.148",
                "port": "",
                "userAgent": "MinIO (linux; amd64) minio-go/v7.0.59 MinIO Console/(dev)"
            },
            "awsRegion": "",
            "eventName": "s3:ObjectCreated:Put",
            "eventTime": "2023-08-28T02:56:20.333Z",
            "eventSource": "minio:s3",
            "eventVersion": "2.0",
            "userIdentity": {
                "principalId": "ROOTNAME"
            },
            "responseElements": {
                "x-amz-id-2": "fad4b3083214c3b0ad28cc0f83fd770a8fd5fb5e47b065bc7cae01b61e817e1a",
                "x-amz-request-id": "177F6C96E763B761",
                "x-minio-deployment-id": "d6a44a90-a62c-4605-8aa2-121a85f0d440",
                "x-minio-origin-endpoint": "http://192.168.192.254:9002"
            },
            "requestParameters": {
                "region": "",
                "principalId": "ROOTNAME",
                "sourceIPAddress": "192.168.16.148"
            }
        }
    ]
}
```

这种格式在mysql中不好查询。因此我使用webhook方式订阅event，这种方式更灵活，我可以提取任何我想要的数据存入数据库。

以下是webhook的方式：

```
docker run \
   -d \
   --network=host \
   --name minio \
   -v /opt/minio/data:/data \
   -e MINIO_ROOT_USER=ROOTNAME \
   -e MINIO_ROOT_PASSWORD=CHANGEME123 \
   -e MINIO_NOTIFY_WEBHOOK_ENABLE_PRIMARY="on" \
   -e MINIO_NOTIFY_WEBHOOK_ENDPOINT_PRIMARY="http://127.0.0.1:8080/buckets/event" \
   quay.io/minio/minio server /data --console-address ":9090" \
   --address=":9002" \
   --ftp="address=:8021" \
   --ftp="passive-port-range=30000-40000"
```

```
mc alias set myminio http://192.168.16.46:9002 ROOTNAME CHANGEME123

mc admin info --json myminio

mc event add myminio/cogent arn:minio:sqs::PRIMARY:webhook --event put

mc event add myminio/cogent arn:minio:sqs::PRIMARY:webhook --event delete

mc event ls myminio/cogent arn:minio:sqs::PRIMARY:webhook
```

> 进入容器后，记得要给 /opt/bin/mc 赋予执行权限

以下是我对minio中object进行删除和添加时收到的json：

删除时，webhook的参数：

```json
{
    "EventName": "s3:ObjectRemoved:Delete",
    "Key": "cogent/jetbra (1).zip",
    "Records": [
        {
            "eventVersion": "2.0",
            "eventSource": "minio:s3",
            "awsRegion": "",
            "eventTime": "2023-08-28T05:55:05.275Z",
            "eventName": "s3:ObjectRemoved:Delete",
            "userIdentity": {
                "principalId": "ROOTNAME"
            },
            "requestParameters": {
                "principalId": "ROOTNAME",
                "region": "",
                "sourceIPAddress": "127.0.0.1"
            },
            "responseElements": {
                "content-length": "160",
                "x-amz-id-2": "fad4b3083214c3b0ad28cc0f83fd770a8fd5fb5e47b065bc7cae01b61e817e1a",
                "x-amz-request-id": "177F765801436093",
                "x-minio-deployment-id": "d6a44a90-a62c-4605-8aa2-121a85f0d440",
                "x-minio-origin-endpoint": "http://192.168.192.254:9002"
            },
            "s3": {
                "s3SchemaVersion": "1.0",
                "configurationId": "Config",
                "bucket": {
                    "name": "cogent",
                    "ownerIdentity": {
                        "principalId": "ROOTNAME"
                    },
                    "arn": "arn:aws:s3:::cogent"
                },
                "object": {
                    "key": "jetbra+%281%29.zip",
                    "sequencer": "177F765801E61109"
                }
            },
            "source": {
                "host": "127.0.0.1",
                "port": "",
                "userAgent": "MinIO (linux; amd64) minio-go/v7.0.59 MinIO Console/(dev)"
            }
        }
    ]
}
```

添加object时，webhook参数

```json
{
    "EventName": "s3:ObjectCreated:Put",
    "Key": "cogent/jetbra (1).zip",
    "Records": [
        {
            "eventVersion": "2.0",
            "eventSource": "minio:s3",
            "awsRegion": "",
            "eventTime": "2023-08-28T05:56:00.485Z",
            "eventName": "s3:ObjectCreated:Put",
            "userIdentity": {
                "principalId": "ROOTNAME"
            },
            "requestParameters": {
                "principalId": "ROOTNAME",
                "region": "",
                "sourceIPAddress": "192.168.16.148"
            },
            "responseElements": {
                "x-amz-id-2": "fad4b3083214c3b0ad28cc0f83fd770a8fd5fb5e47b065bc7cae01b61e817e1a",
                "x-amz-request-id": "177F7664D9E2DAD5",
                "x-minio-deployment-id": "d6a44a90-a62c-4605-8aa2-121a85f0d440",
                "x-minio-origin-endpoint": "http://192.168.192.254:9002"
            },
            "s3": {
                "s3SchemaVersion": "1.0",
                "configurationId": "Config",
                "bucket": {
                    "name": "cogent",
                    "ownerIdentity": {
                        "principalId": "ROOTNAME"
                    },
                    "arn": "arn:aws:s3:::cogent"
                },
                "object": {
                    "key": "jetbra+%281%29.zip",
                    "size": 172138,
                    "eTag": "8943434aec7868e6e16d36209ca47fab",
                    "contentType": "application/x-zip-compressed",
                    "userMetadata": {
                        "content-type": "application/x-zip-compressed"
                    },
                    "sequencer": "177F7664DCA24B86"
                }
            },
            "source": {
                "host": "192.168.16.148",
                "port": "",
                "userAgent": "MinIO (linux; amd64) minio-go/v7.0.59 MinIO Console/(dev)"
            }
        }
    ]
}
```



> 漏洞：如果我们Java程序重启，或者说webhook的web程序重启后，这时我们对minio删除或添加修改对象数据，那么此时将监听不到这个事件。因此我们一种方式是停止web服务时，先停止minio。另一种方式是执行定时任务，同步数据，或者说在web程序启动后一分钟内，同步minio 的object数据。当然如果object非常多，web程序更新频繁，我们可以将minio的event放到MQ中，web程序再去消费MQ中event