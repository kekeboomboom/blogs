# 设计模式---策略模式+工厂

关键词：设计模式，策略模式，工厂模式

## 概要

现在我需要实现一个功能，是添加一路SDI输出，但是输出的协议有不同，有udp、srt等，针对不同的协议我要做不同的操作，后面还有可能添加其他的协议，因此这里面用策略模式不错。

由于单纯的策略模式并不能完全消除if...else...，这里我们用了工厂模式再进行封装（其实就是通过List或Map,消除if...else...）

这里使用springboot管理bean，如果不是spring，自己去new就行。

## 代码概要

### 策略接口

```
public interface SDIStrategy {

    /**
     * 创建SDI，先在数据库中创建此目的地，然后在根据相关协议组装请求，去SMH创建目的地
     */
    void createSDI(Route route, OpenSDIReq req);
}
```

### 策略实现

```
@Component
public class SDIUDPStrategy implements SDIStrategy {

    @Override
    public void createSDI(Route route, OpenSDIReq req) {
        // do something udp一般是内网访问，ip可配
    }
}
```

```java
@Component
public class SDISRTStrategy implements SDIStrategy {

    @Override
    public void createSDI(Route route, OpenSDIReq req) {
		// do something srt给外网用，可配置端口，延时，加密方式，TTL等
    }
}
```

### 工厂

```java
@Component
public class SDIStrategyFactory {

    private static final Map<String, SDIStrategy> strategies = new HashMap<>();

    @Resource
    private SDIUDPStrategy sdiudpStrategy;
    @Resource
    private SDISRTStrategy sdisrtStrategy;

    @PostConstruct
    public void init() {
        strategies.put(SDIProtocolType.UDP.getProtocol(), sdiudpStrategy);
        strategies.put(SDIProtocolType.SRT.getProtocol(), sdisrtStrategy);
    }

    public SDIStrategy getStrategy(String protocol) {
        if (SDIProtocolType.getEnum(protocol) == null) {
            throw new ServiceException("illegal protocol type, please check it");
        }
        return strategies.get(protocol);
    }

}
```

此enum类并不是重点，可不看

```java
public enum SDIProtocolType {

    UDP("udp"),
    SRT("srt");

    private String protocol;

    SDIProtocolType(String protocol) {
        this.protocol = protocol;
    }

    public String getProtocol() {
        return protocol;
    }

    public static SDIProtocolType getEnum(String protocol) {
        for (SDIProtocolType value : SDIProtocolType.values()) {
            if (value.getProtocol().equals(protocol)) {
                return value;
            }
        }
        return null;
    }
}
```



### 最终的调用

```java
@Slf4j
@Service
public class SDIServiceImpl implements ISDIService {
    @Resource
    private SDIStrategyFactory sdiStrategyFactory;
    
    private void createSDIDest(Route route, OpenSDIReq req) {
        String protocol = req.getProtocol();
        SDIStrategy strategy = sdiStrategyFactory.getStrategy(protocol);
        strategy.createSDI(route, req);

    }
}
```

