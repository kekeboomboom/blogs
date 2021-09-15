## 是什么

是一种应用层协议，有html5而推出，是一种全双工的通信协议，不仅客户端可以主动向服务端发送信息，服务端也可以主动向客户端发送信息。

## 为什么

比如更新天气，比如聊天室，如果用ajax去轮询耗时，可能header比较大，所以浪费时间吧。而websocket可以主动推送信息给客户端，头比较小。

## 怎么用

如果是websocket那么http协议header字段upgrade会有websocket标志。

前端js代码，new websocket，send方法发送

后端springboot引入maven以来，配置endpoint，使用注解onmessage，onstart，可以获得链接过来的客户端id。



## 谁再用？

慕课网，我们看到它使用了websocket，

![image-20210825110435074](https://gitee.com/keke518/MarkDownPicture/raw/master/img/20210825110435.png)

他的协议头是wss而不是http，我们看到他用的get方法，网上说他是先通过http请求建立tcp链接，然后再转型成websocket。

返回码看到是`101 Switching Protocols`

与websocket有关的响应头：

![image-20210825141128172](https://gitee.com/keke518/MarkDownPicture/raw/master/img/20210825141128.png)

请求头：

![image-20210825141144599](https://gitee.com/keke518/MarkDownPicture/raw/master/img/20210825141144.png)



慕课网使用websocket发送了那些消息呢？

![image-20210825141502840](https://gitee.com/keke518/MarkDownPicture/raw/master/img/20210825141502.png)

**箭头抄下**的应该是服务器发送到客户端的，里面有pingInterval，这个应该是心跳的功能，每30秒发送一次。你看数据2，3就是心跳，14：04：25然后30秒之后14：04：55又接发了一次2，3 。

pingTimeout应该是心跳超时吧，如果50秒没有心跳，就说明这个链接断掉了。

sid不知道是标识啥的



![image-20210825142154690](https://gitee.com/keke518/MarkDownPicture/raw/master/img/20210825142154.png)

这里面只有url是我当前的浏览器访问的页面，其他字段暂时不知道意味着什么？

![image-20210825142320665](https://gitee.com/keke518/MarkDownPicture/raw/master/img/20210825142320.png)

这条记录我测试出来了，他表示我个人账户的消息

![image-20210825142509896](https://gitee.com/keke518/MarkDownPicture/raw/master/img/20210825142510.png)

之前我这个小铃铛有一条未读消息，然后这个unreads字段就一直是1，然后我看完这个未读消息他就变成0了。

