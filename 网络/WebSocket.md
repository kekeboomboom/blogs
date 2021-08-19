## 是什么

是一种应用层协议，有html5而推出，是一种全双工的通信协议，不仅客户端可以主动向服务端发送信息，服务端也可以主动向客户端发送信息。

## 为什么

比如更新天气，比如聊天室，如果用ajax去轮询耗时，可能header比较大，所以浪费时间吧。而websocket可以主动推送信息给客户端，头比较小。

## 怎么用

如果是websocket那么http协议header字段upgrade会有websocket标志。

前端js代码，new websocket，send方法发送

后端springboot引入maven以来，配置endpoint，使用注解onmessage，onstart，可以获得链接过来的客户端id。

