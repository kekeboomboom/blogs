# mediakit 项目结构

## 概要

项目地址：https://github.com/ZLMediaKit/ZLMediaKit

此项目我们把他做为一个流媒体服务器，我们会有srt和rtsp的流推到mediakit，然后网页使用webrtc播放。

还有使用一些web hook来支持按需推流、不中断录像等，更多功能不再赘述。

虽然我不是专业的流媒体开发人员，但是我们稍微学习一下这个项目，以下是我对这个项目的浅显认识。

## 环境准备

windows11

wsl ubuntu22.04

vscode

## 项目结构

![image-20231108142515362](https://pic-keboom.oss-cn-hangzhou.aliyuncs.com/img/image-20231108142515362.png)

## 第三方库 3rdpart

他这个第三方库是依赖很深的，tcp/udp服务器，各种协议（rtmp，rtsp，hls等）对接都是他们做的，noticeCenter等。

对于这些服务器，如果你要使用http请求，那么你需要创建一个http服务器。如果你要使用rtmp，那么你就创建一个rtmp服务器。只是监听的端口不同。

对于http请求我们都很熟悉，是一请求一响应回答即可。

对于流媒体服务器，比如说我们要实现一个最简的rtmp服务器，首先他仍然是一个tcp服务器，我们监听1935端口。假如我们使用ffmpeg将一个mp4文件推rtmp流到我们rtmp服务器，rtmp服务器收到了数据包，然后我们是有库去解析的，比如ffmpeg库libavcodec，libavformat等等，这些库能讲数据解析成一个格式比如说叫AVFormat这样一个类。接着无论是我们向将他转成rtsp，hls或者其他的什么格式的协议，都是有相应的库去掉用的。比如我们想将他转成rtsp协议推出去，rtspFormatData=rtspLibFunction(AVFormatData, url ) 基本上就是这样一个函数. 这时有rtsp拉流请求，那么rtsp服务器监听到后，看看有没有这个url的数据文件，有就给拉流的播放器。

以上是我的猜想。哈哈，下面是GPT的一个回答：

```
I ask: 
Now i have a rtmp server, server receive a rtmp stream, then convert to rtsp stream to push a url. How to do it ? Show me c++ code.
```

````
GPT answer: 
To design a streaming media server that receives an RTMP stream and converts it to an RTSP stream to push a URL, you can follow the steps below:

1. **Server Architecture**:
   - Create a central server application that listens for incoming RTMP connections.
   - Use a multi-threaded or asynchronous model to handle multiple RTMP connections simultaneously.
   - Consider using a library like libevent or Boost Asio for network operations.

2. **RTMP Server**:
   - Implement an RTMP server class, such as `RtmpServer`, to handle incoming RTMP connections.
   - Use a library like librtmp or RTMPDump to parse and process RTMP packets.
   - Handle RTMP handshake, metadata, and audio/video streams.

3. **RTSP Server**:
   - Implement an RTSP server class, such as `RtspServer`, to handle outgoing RTSP connections.
   - Use a library like live555 or GStreamer to generate RTSP streams.
   - Convert the incoming RTMP stream to an RTSP stream using appropriate codecs and formats.

Here's an example of how the code for the above design may look like in C++:

```cpp
#include <iostream>
#include <thread>
#include <boost/asio.hpp>

class RtmpServer {
public:
    void start() {
        // Start listening for RTMP connections
        // Handle incoming RTMP packets
        // Parse and process handshake, metadata, and audio/video streams
        // Convert RTMP stream to RTSP stream
    }
};

class RtspServer {
public:
    void start() {
        // Start listening for RTSP connections
        // Generate RTSP streams using appropriate codecs and formats
        // Push the RTSP stream to a URL
    }
};

int main() {
    // Create and start the RTMP server
    RtmpServer rtmpServer;
    std::thread rtmpThread([&]() {
        rtmpServer.start();
    });

    // Create and start the RTSP server
    RtspServer rtspServer;
    std::thread rtspThread([&]() {
        rtspServer.start();
    });

    // Wait for the servers to finish
    rtmpThread.join();
    rtspThread.join();

    return 0;
}
```

Please note that the above code is just a basic outline and you would need to implement the specific functionality for RTMP and RTSP handling, as well as handle threading, network operations, error handling, and other details as per your requirements.

It's important to consider that developing a full-fledged streaming media server involves much more complexity and considerations than can be covered in a simple code example. This code serves as a starting point and you may need to use additional libraries, implement error handling, logging, and other features to create a robust and reliable streaming media server.
````

你还可以接着问：
`How to Implement an RTMP server？` 
`And i have a media player , I want get rtmp stream from the rtmp server , how to implementing the RTMP server?`
但是这里就不贴回答了。大家可以自己去GPT~

---

上面说的是tcp服务器和media-server，接着还有一个重要的是ZLToolKit中的noticeCenter，这个是消息中心，比如有新的流推到mediakit或者推流停止、录像开启、录像关闭等等，都会发送通知到noticeCenter，然后这个noticeCenter再将消息发送的监听此消息的listener。

比如说有新的流推到mediakit，那么会向noticeCenter发送流注册消息，那么有个web hook函数就是监听此消息的listener

## api 文件夹

C API，文档说是给嵌入式设备用的

## docker

docker镜像构建，构建之后的镜像大小6百多MB，挺夸张的。

## src

这里面很多涉及音视频协议解析，看不太懂，我只debug了一下，大概说说我看到的。

我用ffmpeg推送一条rtmp流到mediakit，首先rtmp server监听到请求，然后一路执行到Rtmpsession，他接受数据并缓存

