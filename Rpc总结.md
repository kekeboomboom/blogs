> version1.0

**client**：客户端要调用远程方法，需要用RpcClientProxy去动态代理。客户端将服务名，方法名和参数等给代理类。

**simple.RpcClientProxy**：使用JDK动态代理，在invoke方法中将封装RpcRequest。调用sendRpcrequest方法。

**simpe.RpcClient**：实现了sendRpcrequest方法，通过socket发送rpcRequest，并返回result。

**server**：启动server，并新建并注册服务。

**simple.RpcServer**：初始化线程池，创建serverSocket监听客户端连接，如果有新的客户端连接则作为任务在线程池中执行。

**simple.WorkThread**：实现具体的业务逻辑。通过socket接受RpcRequest，然后通过反射调用服务端的服务，然后将调用结果写入socket。RpcClient的sendRpcRequest方法接收到结果，向上return，直到client。

> version2.0

加入Netty，替代socket。使用kryo代替netty自带的编码解码器。

**NettyClientMain**：创建客户端代理类，通过代理类调用远程方法。

**RpcClientProxy**：封装RpcRequest，调用sendRpcRequest方法。

**NettyClientTransport**：实现了sendRpcRequest方法，获得channel，向channel写入RpcRequest，从AttributeKey中获得rpcResponse。

**ChannelProvider**：通过bootstrap监听客户端连接，并且实现了重试机制（通过递归和schedule实现）。

**NettyClientHandler**：channelRead方法，msg即为返回的rpcResponse，将其放入AttributeKey中。

**NettyServer**：初始化netty服务器，配置编码解码器和Handler。

**NettyServerHandler**：channelRead中使用线程池执行任务。msg为rpcRequest，调用rpcRequestHandler.handle获得结果，然后将结果写入channel。

**DefaultServiceRegistry**：使用ConcurrentHashMap作为注册中心。

**ThreadPoolFactory**：提出线程池创建工厂。



> 添加心跳机制

分别在Netty客户端服务端配置心跳。

```java
ch.pipeline().addLast(new IdleStateHandler(0, 5, 0, TimeUnit.SECONDS));
```

然后分别在handler中重写userEventTriggered方法。

```java
    @Override
    public void userEventTriggered(ChannelHandlerContext ctx, Object evt) throws Exception {
        if (evt instanceof IdleStateEvent) {
            IdleState state = ((IdleStateEvent) evt).state();
            if (state == IdleState.WRITER_IDLE) {
                log.info("write idle happen [{}]", ctx.channel().remoteAddress());
                Channel channel = ChannelProvider.get((InetSocketAddress) ctx.channel().remoteAddress());
                RpcRequest rpcRequest = RpcRequest.builder().rpcMessageTypeEnum(RpcMessageTypeEnum.HEART_BEAT).build();
                channel.writeAndFlush(rpcRequest).addListener(ChannelFutureListener.CLOSE_ON_FAILURE);
            }
        } else {
            super.userEventTriggered(ctx, evt);
        }
    }
```

```java
    @Override
    public void userEventTriggered(ChannelHandlerContext ctx, Object evt) throws Exception {
        if (evt instanceof IdleStateEvent) {
            IdleState state = ((IdleStateEvent) evt).state();
            if (state == IdleState.READER_IDLE) {
                log.info("idle check happen, so close the connection");
                ctx.close();
            }
        } else {
            super.userEventTriggered(ctx, evt);
        }
    }
```



> 添加spring注解

