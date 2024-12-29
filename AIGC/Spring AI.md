# Spring AI --- Chat API

看到其他人做，看起来很简单，自己做还是遇到不少问题，所以分几篇写吧。

本篇写，如何在 huggingface 上 deploy 一个 model，然后我们用 spring ai能够调用这个 model 的接口出来结果。



## API 选择

最快能接入的：openAI你得注册，还得小心别被封。当然有各种办法搞定，但是还是繁琐。（前几周我的账号就被封了:sob:)。Azure OpenAI 这个现在注册也要填写各种东西了，要求也越来越严格了。Amazon 我还得看文档去操作，麻烦死了。

本地部署：Ollama 倒是非常方便，可以本地部署，但是我不想在我的笔记本上跑这个东西。我也不想单独去找一个服务器去跑这个，因为有些贵。

最后我选择 HugginFace，他价格便宜（按小时收费），没那么多地区限制，模型众多，部署极其方便。但是整合过程并不顺利。



## huggingface LLMs prepare

### Supported LLMs

[Spring AI API Huggingface](https://docs.spring.io/spring-ai/reference/api/clients/huggingface.html)

Spring AI 支持的 Huggingface 的 text-generation-interface，因此我们必须看此[Text Generation Interface](https://huggingface.co/docs/text-generation-inference/index)。文档中说当前支持的 LLMs：Llama, Falcon, StarCoder, BLOOM, GPT-NeoX, and T5。所以不是你随便选择一个文本生成模型就好哦。



### Deploy a Llama Model

![image-20240429091913880](https://pic-keboom.oss-cn-hangzhou.aliyuncs.com/img/image-20240429091913880.png)



 我们进入到Meta主页，我选择了 llama-2-7b-hf（因为看起来模型小些，服务器会便宜些吧，你也可以试试 llama3）

![image-20240429092250937](https://pic-keboom.oss-cn-hangzhou.aliyuncs.com/img/image-20240429092250937.png)

进入这个页面后，记得往下翻翻他让你留下联系方式才能 access this model，接下来我们使用 Interface endpoint 方式去部署时，需要先access this model



![image-20240429092615472](https://pic-keboom.oss-cn-hangzhou.aliyuncs.com/img/image-20240429092615472.png)

获得到此模型的权限后，我们点击 Interface Endpoints

![image-20240429092725034](https://pic-keboom.oss-cn-hangzhou.aliyuncs.com/img/image-20240429092725034.png)

然后直接点击 create endpoint。



![image-20240429092826497](https://pic-keboom.oss-cn-hangzhou.aliyuncs.com/img/image-20240429092826497.png)

然后我们可以开启和暂停此 Endpoint，在暂停期间是不收费的。



### Token

![image-20240429093016469](https://pic-keboom.oss-cn-hangzhou.aliyuncs.com/img/image-20240429093016469.png)



token 自己申请一个 read token 即可。



## Create Spring AI project

### pom.xml

![image-20240429103532122](https://pic-keboom.oss-cn-hangzhou.aliyuncs.com/img/image-20240429103532122.png)



### Test API

```java
    @Test
    void callFace() {
        HuggingfaceChatClient huggingfaceChatClient = new HuggingfaceChatClient("your_token",
                "https://api-inference.huggingface.co/models/meta-llama/Llama-2-7b-chat-hf");
        String mistral7bInstruct = """
        [INST] You are a helpful code assistant. Your task is to generate a valid JSON object based on the given information:
        name: John
        lastname: Smith
        address: #1 Samuel St.
        Just generate the JSON object without explanations:
        [/INST]""";
        Prompt prompt = new Prompt(mistral7bInstruct);
        ChatResponse aiResponse = huggingfaceChatClient.call(prompt);
        System.out.println(aiResponse.getResult().getOutput().getContent());
    }
```



Ok, 这样我们就能够调用成功了。Congratulation~



---



接下来，我的文章会写如何将一个 pdf 放到向量数据库中。
