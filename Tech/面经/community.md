# 论坛项目



几个主要功能点，以此来应对面试提问吧

## 使用GitHub，gitee实现登录功能

[GitHub文档](https://docs.github.com/en/developers/apps/building-oauth-apps/authorizing-oauth-apps)

![image-20210729202656126](https://gitee.com/keke518/MarkDownPicture/raw/master/img/image-20210729202656126.png)

### 1.首先我们向GitHub发送授权请求

![image-20210729200937997](https://gitee.com/keke518/MarkDownPicture/raw/master/img/image-20210729200937997.png)

`client_id`是必须的，我们在GitHub上创建OAuth Apps，然后就会获得。

`redirect_uri`重定向地址

```html
                    <a th:href="@{https://github.com/login/oauth/authorize(client_id='7f316909bf70d1eaa2b2',redirect_uri=${#httpServletRequest.getServletContext().getAttribute('githubRedirectUri')},scope='user',state=1)}">
                        <img src="/images/github.png">
                    </a>
```

![image-20210729203917347](https://gitee.com/keke518/MarkDownPicture/raw/master/img/image-20210729203917347.png)

回调地址是我们自己的后端：

```java
    @GetMapping("/callback/{type}")
    public String newCallback(@PathVariable(name = "type") String type,
                              @RequestParam(name = "code") String code,
                              @RequestParam(name = "state", required = false) String state,
                              HttpServletRequest request,
                              HttpServletResponse response) {
        UserStrategy userStrategy = userStrategyFactory.getStrategy(type);
        LoginUserInfo loginUserInfo = userStrategy.getUser(code, state);
        if (loginUserInfo != null && loginUserInfo.getId() != null) {
            User user = new User();
            String token = UUID.randomUUID().toString();
            user.setToken(token);
            user.setName(loginUserInfo.getName());
            user.setAccountId(String.valueOf(loginUserInfo.getId()));
            user.setType(type);
            UFileResult fileResult = null;
            try {
                fileResult = uFileService.upload(loginUserInfo.getAvatarUrl());
                user.setAvatarUrl(fileResult.getFileUrl());
            } catch (Exception e) {
                user.setAvatarUrl(loginUserInfo.getAvatarUrl());
            }
            userService.createOrUpdate(user);
            Cookie cookie = new Cookie("token", token);
            cookie.setMaxAge(60 * 60 * 24 * 30 * 6);
            cookie.setPath("/");
            response.addCookie(cookie);
            return "redirect:/";
        } else {
            log.error("callback get github error,{}", loginUserInfo);
            // 登录失败，重新登录
            return "redirect:/";
        }
    }
```

这里用了策略模式，那么我们看到`LoginUserInfo loginUserInfo = userStrategy.getUser(code, state);`

进行追踪：`GithubUserStrategy`-`GithubProvider`

![image-20210729204948043](https://gitee.com/keke518/MarkDownPicture/raw/master/img/image-20210729204948043.png)

进入`getAccessToken`，可以看到通过okhttp3框架发送post请求来获得token

拿到token后调用getUser方法，向GitHub发送get请求头Authorization要带上token，然后就获得了用户的信息了，如用户的名字，头像url等。

## 热门标签

从数据库的question表中取前20的数据，然后获得他们的tags（tags之间用，分割所以需要split），然后将这些tags放到map中，value为权重，然后构造dto实现compare方法，放到优先级队列中即可排好顺序。这也是我们所说的topN问题。

## 提问功能

也就是发帖子功能，使用了一个插件Editor.md可以编辑markdown，使用ucloud云服务，将头像图片或者帖子中的图片放到云服务器上。

