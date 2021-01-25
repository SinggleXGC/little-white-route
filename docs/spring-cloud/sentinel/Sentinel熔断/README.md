# Sentinel熔断



## 自定义降级方法

```java
@RequestMapping("/consumer/fallback/{id}")
@SentinelResource(value = "fallback", fallback = "handleFallBack")
public String fallback(@PathVariable String id) {
    return "fallback";
}

public String handleFallBack(@PathVariable String id, Throwable e) {
    return "handle fallback";
}
```



## 自定义限流方法

```java
@RequestMapping("/consumer/limit/{id}")
@SentinelResource(value = "limit", blockHandler = "handleLimit")
public String limit(@PathVariable String id) {
    return "fallback";
}

public String handleLimit(@PathVariable String id, BlockException blockException) {
    return "handle fallback";
}
```



## fallback和blockHandler同时配置

限流且报错，使用blockHandler



## 忽略指定异常

```java
@RequestMapping("/consumer/fallback/{id}")
@SentinelResource(value = "fallback", fallback = "handleFallBack", 
                  exceptionsToIgnore = {IllegalArgumentException.class})
public String fallback(@PathVariable String id) {
    return "fallback";
}
```



## 结合Feign完成服务降级(消费侧配置)

1. 增加依赖

   ```xml
   <dependency>
       <groupId>org.springframework.cloud</groupId>
       <artifactId>spring-cloud-starter-openfeign</artifactId>
   </dependency>
   ```

2. yml配置

   ```yml
   #激活sentinel对Feign的支持
   feign:
     sentinel:
       enabled: true
   ```

3. 主启动类增加注解`@EnableFeignClients`

4. 增加远程service

   ```java
   @FeignClient(value = "nacos-payment-provider", fallback = PaymentFallbackService.class)
   public interface PaymentService {
   
       @GetMapping("/testA")
       public String testA();
   }
   ```

5. 增加fallback方法

   ```java
   @Component
   public class PaymentFallbackService implements PaymentService{
       @Override
       public String testA() {
           return "testA fallback";
       }
   }
   ```

   

