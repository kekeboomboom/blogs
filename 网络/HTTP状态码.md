我使用的是新版Edge浏览器，右键，点击检查，点击网络，可以看到请求的各种文件。那么以此来看看状态码的使用吧。

### 101

与websocket相关，[websocket在慕课网中的应用 - KeBoom - 博客园 (cnblogs.com)](https://www.cnblogs.com/keboom/p/15184913.html)

### 200

是绝大多数的响应码，表示请求成功

### 204

####  简书

> 请求 URL: https://www.jianshu.com/shakespeare/notes/dd8285b01b48/mark_viewed
>
> 请求方法: POST
>
> 状态代码: 204 No Content

根据url地址，可以推断出应该是标记当前文章**我**看过了。

还有就是此为post请求，并且Content-Type: application/json  表单数据变为了请求负载

内容为：{"fuck":1}

为什么是请求负载而不是表单呢？相关回答：[StackOverflow](https://stackoverflow.com/questions/23118249/whats-the-difference-between-request-payload-vs-form-data-as-seen-in-chrome#:~:text=The%20Request%20Payload%20-%20or%20to%20be%20more,headers%20and%20the%20CRLF%20of%20a%20HTTP%20Request.)

我理解的意思就是，它使用了ajax来发送post请求，并且content-type为json，这样的话，数据将为请求负载而不是表单数据。

#### 知乎

发起预检options请求，我所知道的option请求是来解决跨域问题的。先发送options请求来看看允许接受我的哪些方法（比如get，post，put等等）允许接受哪些头（比如Authorization, Content-Type, X-API-Version等等），然后我们在发送比如get请求去获得资源。

那么知乎使用预检请求，返回状态码204

知乎还有一个请求 https://www.zhihu.com/sc-profiler 为post，数据在请求负载（[["i","production.heifetz-column.desktop.all.column.FetchErrorV2.CrossOrigin.https-zhuanlan-zhihu-com.GET.https-www-zhihu-com.H_6.unlimited-vip_rights-popup.count",1,1]]）中，响应码为204，我猜测也是用来分析用户行为的。



### 302

#### CSDN

我使用的QQ登陆的csdn，请求我qq图片时响应码为302

请求 URL: https://profile.csdnimg.cn/7/A/5/3_qq_27541519看到这个url应该是使用我qq登录时，将我的qq头像存储在csdn的图片服务器中，然后获取我的头像时去重定向到图片服务器中的头像。那我的qq头像可能会更换，那时就需要重新更改头像url地址，所以才使用302表示临时重定向吧。



### 304

#### 简书

我在第一次访问某篇帖子，这个帖子的作者的最新笔记，建议阅读，音频等等信息会用get请求获取，而这些信息基本是没那么容易变的，对于同一个作者来说这些信息是固定的。第一次访问返回响应码为200

那么我刷新页面，这时我们会看到有很多响应码为304的表示此作者的这些信息没有修改，那么就使用浏览器缓存的数据。



![image-20210825143841899](https://gitee.com/keke518/MarkDownPicture/raw/master/img/20210825143842.png)

可以看到很多304字段的body大小都是0，说明使用的是浏览器缓存的数据，服务端没有传数据过来。注意看这个url，我在清除缓存然后刷新页面，那么这个url的响应码就是200了：

![image-20210825144232274](https://gitee.com/keke518/MarkDownPicture/raw/master/img/20210825144232.png)

可以看到他的响应码为200，并且body为73

