# SpringBoot WebSocket STOMP

关键词：Springboot, WebSocket, STOMP, broadcast, sendToUser, MessageMapping, SubscribeMapping, convertAndSendToUser



STOMP是一种发布订阅的模式，被订阅者发布消息以广播形式发送。如果需要一对一发送或者说指定某个客户端发送，springboot提供了convertAndSendToUser方法去指定user进行发送。

本文实现了既有广播形式，也有指定user发送形式，以做对比。



[代码参考](https://github.com/spring-guides/gs-messaging-stomp-websocket)

## maven

```
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-websocket</artifactId>
        </dependency>
```

## WebSocketConfig

```
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.messaging.simp.config.MessageBrokerRegistry;
import org.springframework.web.socket.config.annotation.EnableWebSocketMessageBroker;
import org.springframework.web.socket.config.annotation.StompEndpointRegistry;
import org.springframework.web.socket.config.annotation.WebSocketMessageBrokerConfigurer;
import org.springframework.web.socket.server.standard.ServletServerContainerFactoryBean;

/**
 * {@code @author:} keboom
 * {@code @date:} 2023/9/21
 * {@code @description:}
 */
@Configuration
@EnableWebSocketMessageBroker
public class WebSocketConfig implements WebSocketMessageBrokerConfigurer {

    @Override
    public void configureMessageBroker(MessageBrokerRegistry config) {
        config.enableSimpleBroker("/topic");
        config.setApplicationDestinationPrefixes("/app");
    }

    @Override
    public void registerStompEndpoints(StompEndpointRegistry registry) {
        registry.addEndpoint("/mobicaster-websocket/{androidID}").setAllowedOrigins("*").
        setHandshakeHandler(new CustomHandshakeHandler());
    }

    @Bean
    public ServletServerContainerFactoryBean createWebSocketContainer() {
        ServletServerContainerFactoryBean container = new ServletServerContainerFactoryBean();
        container.setMaxTextMessageBufferSize(8192);
        container.setMaxBinaryMessageBufferSize(8192);
//        container.setMaxSessionIdleTimeout(10000L);
        return container;
    }
}
```

`registry.addEndpoint("/mobicaster-websocket/{androidID}")`这个是网页向服务器开启一个websocket连接的url地址。{androidID} （这个名字大家随便起哈，随便叫devId，sessionId都行）是作为一个websocket标识，这样我们服务器想要主动向一个websocket客户端发送message时，可以知道应该向谁发。

`config.setApplicationDestinationPrefixes("/app");`这个是网页向服务器发送消息的uri前缀

`config.enableSimpleBroker("/topic");`这个是服务器向网页发送消息的 uri 的“前缀”

`setHandshakeHandler(new CustomHandshakeHandler());`顾名思义，这是websocket握手时，做一些自定义处理的handler。

`ServletServerContainerFactoryBean`这个大家根据需求配了。setMaxSessionIdleTimeout这个在网页上的表示时，如果在一定时期没有发送任何消息，那么当前连接断开，然后建立一个新链接。

## CustomHandshakeHandler

在这个类里面，我们对每个websocket做标识，标识每个websocket的user是什么，到时候服务器主动发送message，将根据user去发。

```java
import org.springframework.http.server.ServerHttpRequest;
import org.springframework.web.socket.WebSocketHandler;
import org.springframework.web.socket.server.support.DefaultHandshakeHandler;
import java.security.Principal;
import java.util.Map;

/**
 * {@code @author:} keboom
 * {@code @date:} 2023/9/22
 * {@code @description:}
 */
public class CustomHandshakeHandler extends DefaultHandshakeHandler {

    @Override
    protected Principal determineUser(ServerHttpRequest request, WebSocketHandler wsHandler, Map<String, Object> attributes) {
        String uri = request.getURI().toString();
        String androidID = uri.substring(uri.lastIndexOf("/") + 1);
        return () -> androidID;
    }

}
```

## Interception

如果你想对websocket的 Connect、subscribe、heartbeat、send、disconnect等事件进行拦截，来做一些处理的话，可以使用拦截器。

这里我需要对disconnect事件进行拦截，来通知另一个程序某个websocket断开连接了。

代码如下：

```java
import jakarta.annotation.Resource;
import org.springframework.messaging.Message;
import org.springframework.messaging.MessageChannel;
import org.springframework.messaging.simp.stomp.StompCommand;
import org.springframework.messaging.simp.stomp.StompHeaderAccessor;
import org.springframework.messaging.support.ExecutorChannelInterceptor;
import org.springframework.web.client.RestTemplate;

import java.security.Principal;


/**
 * {@code @author:} keboom
 * {@code @date:} 2023/9/26
 * {@code @description:}
 */
public class MyChannelInterceptor implements ExecutorChannelInterceptor {

//    @Resource
//    private RestTemplate restTemplate;

    @Override
    public Message<?> preSend(Message<?> message, MessageChannel channel) {
        StompHeaderAccessor accessor = StompHeaderAccessor.wrap(message);
        StompCommand command = accessor.getCommand();
        // 告知FC设备离线了
        if (StompCommand.DISCONNECT.equals(command)) {
            Principal simpUser = (Principal) message.getHeaders().get("simpUser");
            System.out.println("androidID: " + simpUser.getName() + " is offline");
//            restTemplate.getForObject("http://localhost:8081/offline/" + simpUser.getName(), String.class);
        }
        return message;
    }
}
```

debug，查看message里面有什么：

![image-20230927115646774](https://pic-keboom.oss-cn-hangzhou.aliyuncs.com/img/image-20230927115646774.png)

可以看到simpUser有我们想要的值，就是每个websocket的url末尾的用于标识每个websocket连接的androidID。通过这个androidID来知道哪个websocket断开连接了。

[spring 官方关于 Interception](https://docs.spring.io/spring-framework/reference/web/websocket/stomp/interceptors.html)

## Controller

接下来看看我们的controller：

```java
import com.cogent.mobicasterserver.controller.vo.DeviceInfo;
import com.cogent.mobicasterserver.controller.vo.FoldbackReq;
import com.cogent.mobicasterserver.controller.vo.LiveReq;
import com.cogent.mobicasterserver.controller.vo.RespVO;
import jakarta.annotation.Resource;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.messaging.handler.annotation.MessageMapping;
import org.springframework.messaging.handler.annotation.Payload;
import org.springframework.messaging.handler.annotation.SendTo;
import org.springframework.messaging.simp.SimpMessagingTemplate;
import org.springframework.messaging.simp.annotation.SendToUser;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

import static org.springframework.web.bind.annotation.RequestMethod.GET;

/**
 * {@code @author:} keboom
 * {@code @date:} 2023/9/21
 * {@code @description:}
 */
@RestController
public class MobiController {

    private SimpMessagingTemplate template;

    @Autowired
    public MobiController(SimpMessagingTemplate template) {
        this.template = template;
    }

    @RequestMapping(path = "/foldback", method = GET)
    public void foldback(FoldbackReq req) {
        this.template.convertAndSendToUser(req.getAndroidID(), "/topic/foldback", req.toString());
    }


    @MessageMapping("/greetings")
    @SendTo("/topic/greetings")
    public RespVO greetings(@Payload LiveReq liveReq) throws Exception {
		// do something ....
        return new RespVO(200, "success", null);
    }

    @MessageMapping("/live")
    @SendToUser("/topic/live")
    public RespVO live(@Payload LiveReq liveReq) throws Exception {
		// do something ....
        return new RespVO(200, "success", null);
    }
}
```

网上很多资料写的用 @Controller，我这边用的@RestController 完全没问题。

对于 sendToUser的，uri前缀需要加 /user ，这个通过下面的网页端 js 代码更清晰，还有就是我们看浏览器开发者工具的具体websocket的message更清楚，这里就不说每个注解的意思了。

## app.js

```javascript
const stompClient = new StompJs.Client({
    brokerURL: 'ws://localhost:8082/mobicaster-websocket/androidId1234'
});

// --------------------------------------------------------------------------------------------
const stompClient2 = new StompJs.Client({
    brokerURL: 'ws://localhost:8082/mobicaster-websocket/androidId2345'
});
stompClient2.onConnect = (frame) => {
    stompClient2.subscribe('/topic/greetings', (greeting) => {
        showGreeting(JSON.parse(greeting.body).content);
    });
    stompClient2.subscribe('/user/topic/live', (greeting) => {
        // showGreeting(JSON.parse(greeting.body).content);
        console.log('Live: ' + greeting);
    });

    stompClient2.subscribe('/user/topic/foldback', (greeting) => {
        // showGreeting(JSON.parse(greeting.body).content);
        console.log('foldback: ' + greeting);
    });
};
stompClient2.onWebSocketError = (error) => {
    console.error('Error with websocket', error);
};

stompClient2.onStompError = (frame) => {
    console.error('Broker reported error: ' + frame.headers['message']);
    console.error('Additional details: ' + frame.body);
};
// --------------------------------------------------------------------------------------------
stompClient.onConnect = (frame) => {
    setConnected(true);
    console.log('Connected: ' + frame);
    stompClient.subscribe('/topic/greetings', (greeting) => {
        showGreeting(JSON.parse(greeting.body).content);
    });

    stompClient.subscribe('/user/topic/live', (greeting) => {
        // showGreeting(JSON.parse(greeting.body).content);
        console.log('Live: ' + greeting);
    });

    stompClient.subscribe('/user/topic/foldback', (greeting) => {
        // showGreeting(JSON.parse(greeting.body).content);
        console.log('foldback: ' + greeting);
    });
};

stompClient.onWebSocketError = (error) => {
    console.error('Error with websocket', error);
};

stompClient.onStompError = (frame) => {
    console.error('Broker reported error: ' + frame.headers['message']);
    console.error('Additional details: ' + frame.body);
};

function setConnected(connected) {
    $("#connect").prop("disabled", connected);
    $("#disconnect").prop("disabled", !connected);
    if (connected) {
        $("#conversation").show();
    } else {
        $("#conversation").hide();
    }
    $("#greetings").html("");
}

function connect() {
    stompClient.activate();
    stompClient2.activate();
}

function disconnect() {
    stompClient.deactivate();
    stompClient2.deactivate();
    setConnected(false);
    console.log("Disconnected");
}

function sendName() {
    stompClient.publish({
        destination: "/app/greetings",
        body: JSON.stringify({'androidId': "abad123svbasd12", 'liveAction': "start"})
    });

    stompClient.publish({
        destination: "/app/live",
        body: JSON.stringify({'androidId': "abad123svbasd12", 'liveAction': "start"})
    });

    stompClient2.publish({
        destination: "/app/greetings",
        body: JSON.stringify({'androidId': "abad123svbasd12", 'liveAction': "start"})
    });

    stompClient2.publish({
        destination: "/app/live",
        body: JSON.stringify({'androidId': "abad123svbasd12", 'liveAction': "start"})
    });
}

function showGreeting(message) {
    $("#greetings").append("<tr><td>" + message + "</td></tr>");
}

$(function () {
    $("form").on('submit', (e) => e.preventDefault());
    $("#connect").click(() => connect());
    $("#disconnect").click(() => disconnect());
    $("#send").click(() => sendName());
});
```

这里我发起了两个连接，主要对比广播和一对一的user发送的效果。

## index.html

```html
<!DOCTYPE html>
<html>
<head>
    <title>Hello WebSocket</title>
    <link rel="stylesheet" href="https://maxcdn.bootstrapcdn.com/bootstrap/3.3.7/css/bootstrap.min.css" integrity="sha384-BVYiiSIFeK1dGmJRAkycuHAHRg32OmUcww7on3RYdg4Va+PmSTsz/K68vbdEjh4u" crossorigin="anonymous">
    <link href="/main.css" rel="stylesheet">
    <script src="https://code.jquery.com/jquery-3.1.1.min.js"></script>
    <script src="https://cdn.jsdelivr.net/npm/@stomp/stompjs@7.0.0/bundles/stomp.umd.min.js"></script>
    <script src="/app.js"></script>
</head>
<body>
<noscript><h2 style="color: #ff0000">Seems your browser doesn't support Javascript! Websocket relies on Javascript being
    enabled. Please enable
    Javascript and reload this page!</h2></noscript>
<div id="main-content" class="container">
    <div class="row">
        <div class="col-md-6">
            <form class="form-inline">
                <div class="form-group">
                    <label for="connect">WebSocket connection:</label>
                    <button id="connect" class="btn btn-default" type="submit">Connect</button>
                    <button id="disconnect" class="btn btn-default" type="submit" disabled="disabled">Disconnect
                    </button>
                </div>
            </form>
        </div>
        <div class="col-md-6">
            <form class="form-inline">
                <div class="form-group">
                    <label for="name">What is your name?</label>
                    <input type="text" id="name" class="form-control" placeholder="Your name here...">
                </div>
                <button id="send" class="btn btn-default" type="submit">Send</button>
            </form>
        </div>
    </div>
    <div class="row">
        <div class="col-md-12">
            <table id="conversation" class="table table-striped">
                <thead>
                <tr>
                    <th>Greetings</th>
                </tr>
                </thead>
                <tbody id="greetings">
                </tbody>
            </table>
        </div>
    </div>
</div>
</body>
</html>
```

## 查看浏览器，打开开发者工具

### 点击connect

![image-20230925111630894](https://pic-keboom.oss-cn-hangzhou.aliyuncs.com/img/image-20230925111630894.png)

查看这两个websocket的url，查看message订阅的uri。

/user/topic/live和/user/topic/foldback是指定user进行发送的。

/topic/greetings是进行广播。

### 点击send

![image-20230925112009004](https://pic-keboom.oss-cn-hangzhou.aliyuncs.com/img/image-20230925112009004.png)

根据上面 app.js 的代码，或者我们从message中也可以看到，我们向服务器send ：/app/greetings和 /app/live。

我们根据controller的代码，看到/app/greetings发送到此方法：

```java
    @MessageMapping("/greetings")
    @SendTo("/topic/greetings")
    public RespVO greetings(@Payload LiveReq liveReq) throws Exception {
        return new RespVO(200, "success", null);
    }
```

SendTo注解，意思是广播。

```java
    @MessageMapping("/live")
    @SendToUser("/topic/live")
    public RespVO live(@Payload LiveReq liveReq) throws Exception {

        return new RespVO(200, "success", null);
    }
```

SendToUser注解，则标识只向此发送 /app/live 的websocket 返回 /user/topic/live

在 app.js代码中，我们的两个websocket连接分别向服务器发送了 /app/greetings和 /app/live ，因此我们可以看到这两个websocket分别接受到了两个/topic/greetings和一个/user/topic/live。说明 /topic/greetings 确实是广播，/user/topic/live确实一对一的。

### 主动向浏览器发送message

```java
    private SimpMessagingTemplate template;

    @Autowired
    public MobiController(SimpMessagingTemplate template) {
        this.template = template;
    }

    @RequestMapping(path = "/foldback", method = GET)
    public void foldback(FoldbackReq req) {
        this.template.convertAndSendToUser(req.getAndroidID(), "/topic/foldback", req.toString());
    }
```

![image-20230925113226727](https://pic-keboom.oss-cn-hangzhou.aliyuncs.com/img/image-20230925113226727.png)

接着查看网页上的两个websocket：
![image-20230925113329829](https://pic-keboom.oss-cn-hangzhou.aliyuncs.com/img/image-20230925113329829.png)

![image-20230925113404088](https://pic-keboom.oss-cn-hangzhou.aliyuncs.com/img/image-20230925113404088.png)

可以看到只有其中对应的websocket接受到了/user/topic/foldback。



## other

### SubscribeMapping

https://medium.com/swlh/websockets-with-spring-part-3-stomp-over-websocket-3dab4a21f397

此注解当客户端发起订阅后，服务器就立刻发送message给客户端，这种一次性的，主要用来做初始化操作。

### endpoint

多个endpoint，虽然在java上可以进行配置，但是网络上并没有看到对此更详细的使用。



参考资料：

https://spring.io/guides/gs/messaging-stomp-websocket/

https://docs.spring.io/spring-framework/reference/web/websocket/stomp/overview.html

