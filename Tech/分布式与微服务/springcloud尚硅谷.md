2020/5/5 创建消费者和提供者，使用eureka做注册中心。

## OpenFeign

### 使用步骤：

1. 引入依赖

    ```xml
            <!--openfeign-->
            <dependency>
                <groupId>org.springframework.cloud</groupId>
                <artifactId>spring-cloud-starter-openfeign</artifactId>
            </dependency>
    ```

2. 启动类

    ```java
    @SpringBootApplication
    @EnableFeignClients
    public class OrderFeignMain80 {
        public static void main(String[] args) {
            SpringApplication.run(OrderFeignMain80.class, args);
        }
    }
    ```

3. 暴露接口

    ```java
    @Component
    @FeignClient(value = "CLOUD-PAYMENT-SERVICE")
    public interface PaymentFeignService {
    
        @GetMapping(value = "/payment/get/{id}")
        public CommonResult<Payment> getPaymentById(@PathVariable("id") Long id);
    
        @GetMapping(value = "/payment/feign/timeout")
        public String paymentFeignTimeout();
    }
    ```

    `CLOUD-PAYMENT-SERVICE`是`PaymentMain8001`和`8002`的服务名。下面的方法就是原封不动的`getPaymentById`控制器的方法。

4. 自己去调用

    ```java
    @RestController
    @Slf4j
    public class OrderFeignController {
        @Resource
        private PaymentFeignService paymentFeignService;
    
        @GetMapping(value = "/consumer/payment/get/{id}")
        public CommonResult<Payment> getPaymentById(@PathVariable("id") Long id){
            return paymentFeignService.getPaymentById(id);
        }
    }
    ```




## Hystrix

- 服务降级：如果调用超时，或者出错则进入fallback方法

- 服务熔断：

    ![image-20210509192509417](https://gitee.com/keke518/MarkDownPicture/raw/master/20210509192515.png)

    如果出错则进入fallback，然后如果10次请求中有60%都失败了，则熔断（即使正确调用，也是返回错误），然后一段时间后，错误率下降了，则进入半熔断状态，最后则进入正常状态。





## Gateway

### 为什么用gateway

springcloud Gateway是异步非阻塞，webflux+netty。zuul是阻塞。

目前我觉得主要功能就是过滤器。匹配请求做router，过滤请求，增强请求和相应。





## stream

rabbitmq的消息重复和持久化，group属性至关重要！！！