# claude code practice

工具只是作为工具，关键在于使用者的思维方式。如果你把他当做一个只会写写增减某个字段的基础工作者，那么他就只是一个基础工作者。但如果你把他当成一个能够进行市场调研、需求分析、代码设计、代码执行、检验效果等等的各种角色的话，那么他就是会这些角色。

很简单的，如果还是把 AI 当做一种文章总结工具，那么他也就是总结工具了。但是如果你在思考他作为某个人类角色，他应该做什么时，那么他其实就是你的“伙伴”了。这一点我在创建 claude code 的 subagent 时慢慢有所体会。

## 怎么用

我是用的拼车，Claude code Max 20X，六人车，每个人 398 软妹币

## 插件

插件最大的好处就是很方便引用代码块，而且我觉得能够引用代码块这件事很重要，你引用的代码块越精准，AI 才能更好的理解你要做什么。

cursor 上 claude code 插件在 cursor 上是旧版本，需要从 vscode marketplace 下载 visx 文件

但看起来cursor 上装这个插件有 bug，开启 claude 后，一段时间插件会显示 ide 会断开连接。但是 IDEA 上插件并不会。

## cursor vs cc

> 下文中，cc 表示 claude code
> 

我使用 cursor 有一段时间，所以我打算用 cursor 里面的一些概念来类比。

### cc 的 subagent

subagent. cursor 中没有 subagent 概念。subagent 的上下文与 main agent 分离。cc 发明 subagent 是为了防止 main agent 上下文太长的问题。比如我的 main agent 想做的是查询当前需求相关的文件，分析文件，生成计划，执行计划。但是这样 main agent 上下文会太长，并且会有很多与任务无关的内容，扰乱 AI 的判断。比如AI 查找相关文件时，发现了三个文件，但是其中两个文件是无关的，然后你告诉 AI 说着两个文件没用，应该看第三个文件。那么这种排除错误答案的信息，其实对于最终目标来说就是无用的，对于 main agent 来说第三个正确文件才是有用的信息。

有了 subagent 我们就可以在 main agent “持续”的做事，而不是频繁 new chat

> **这对与 cursor 倒是一个启发。我是否可以指定 cursor 用哪个模型，去完成什么样的任务呢？**

cursor中有一个设置，setting→chat→custom model 这个选项打开，这个选项中可以定义要做什么事情。但是这也只是提前定义了 model 和一些 prompt 而已，并不是那种树状的 main agent 和 subagent 的关系
> 

我是怎么用subagent？

我创建了三个角色，feature analyzer, tech leader, fullstack-executor。

feature analyzer来分析历史模块代码生成功能文档，文档大致描述这个feature 是做什么的，有哪些主要的类，数据flow 等。这个文档需要我们去 check 哪些部分是需要的，哪些部分是错误的。这个文档可能会不断膨胀，我们需要定时瘦身。

tech leader 根据上一个 subagent 生成的历史文档和当前需要加的新需求，生成一个技术文档。给出需要改写哪些代码，业务逻辑是什么，最终文件结尾是一个 todo list。这个文档每次做新需求的时候都需要 check，我现在是每次一个新需求，就根据日期生成一个文件夹，这一次的需求的所有 doc 都放到这个文件夹中。tech leader 一次性生成的文档，大概率有一部分不是你想象中要的，因为你就那几句 prompt，让人家生成几百行的 doc，还让人家一次性写对，肚子里的蛔虫也不过如此了吧～

fullstack-executor 根据tech leader任务文档一步步执行任务，完成了就打勾。实时调控方向。

需求完成后，再让feature analyzer根据 tech leader 生成的执行文档，来更新feature analyzer自己生成的文档。这样历史上做的所有新功能都能涵盖到，AI 能获得这个模块的所有信息。

最近又加了一个 code-revert subagent，因为很多时候AI 改的代码不是我们想要的，无论是因为我们 prompt 模糊、错误，还是因为 AI 本身出错，我们都希望回退代码。有了这个 subagent，可以方便我们回退代码，并且不占用 main agent context。我觉得这个功能应该 cc 来做，而不是用户做。

### slash command

这个可以类比 cursor 中的 mdc 文件，提前写好一些 prompt，方便以后直接引用。

这个 slash command 其实就是提前定义了一些 prompt，方便用户调用。我看到不少开源项目都定义了这个 slash command，但我实在有点不感冒，没觉得那些 slash command 很惊艳，非用不可的程度。

有趣的是，这个 slash command 是可以调用 subagent 的。但是我现在还没有遇到可以百分百信任AI，不需要与用户交互的场景。基本上都需要人类自己 check 结果的。

### memory

cc 中是 [CLAUDE.md](http://CLAUDE.md) 文件，cursor 中自带了memory 和 index 功能。augment 也有 memory 这个文件。我觉得 memory 无论是对于写出正确的代码，或者符合开发者个人品味的代码都很重要，但是看起来 cc 做的并不好。很多人说 CLAUDE.md 文件根本没有起作用。

### 前端能力

cursor 更胜一筹，正确率更高。

### 易用度

感觉 cursor 比 cc 更 agentic，augment 比 cursor 更 agentic。我认为未来一定是“傻瓜化”才是对的。用户不应该知道太多细节，不需要知道太多细节，应该做的非常好用，非常符合直觉，非常普世化才对。手机的操作是越来越简化的，空调、电视遥控器是越来越简化的，汽车、电车的操控是越来越简化的。claude 的模型即使可以把代码写的很好，但是他不会成为一个很好的产品推而广之。真希望 ChatGPT 能够推出一款编程 IDE，他一定能够做出一个非常好的产品。

大家现在吹 cc，完全是因为claude 账号的 token 更“便宜”。如果让 cursor 拿到更便宜的 token，谁会去用一个破命令行呢？

cc 除了一个 subagent 是一个新概念，其他的都是旧概念。但实际上 subagent 所谓独立 context ，不占用 main agent context，我认为也没啥用，因为每个 subagent 写的结果我都需要 check，这跟开一个新 chat 没区别呀。

## 薅羊毛

cc 目前 token 很”便宜“，就像现在的淘宝闪购，大家先薅羊毛，等他们不补贴了，再跑路就好了。哈哈

我今天用淘宝闪购红包，吃了个一分钱的外卖。