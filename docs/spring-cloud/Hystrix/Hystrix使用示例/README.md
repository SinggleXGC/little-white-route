# Hystrix使用示例

## Hystrix服务降级配置

Hystrix的服务降级可以在服务提供者侧使用，也可以在服务消费者侧使用。



### 服务提供侧配置

#### 前置准备

一个微服务



#### 添加依赖

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-hystrix</artifactId>
</dependency>
```



#### 开启断路器功能

在主启动类增加`@EnableCircuitBreaker`注解

```java
@SpringBootApplication
@EnableEurekaClient
@EnableCircuitBreaker
public class PaymentHystrixApplication {

    public static void main(String[] args) {
        SpringApplication.run(PaymentHystrixApplication.class, args);
    }
}
```



#### service层方法配置

在Controller调用的service方法上标注@HystrixCommand注解，执行并编写对应的备选响应方法。

@HystrixCommand注解的fallbackMethod属性指定备选响应方法，commandProperties配置断路器的属性。`execution.isolation.thread.timeoutInMilliseconds`属性指定多少秒没有返回响应就返回备选响应方法。

```java
@HystrixCommand(fallbackMethod = "paymentInfo_TimeoutHandler", commandProperties = {
        @HystrixProperty(name = "execution.isolation.thread.timeoutInMilliseconds", value = "3000")
})
public String paymentInfo_Timeout(Integer id) {
    int timeUnit = 4;
    try {
        TimeUnit.SECONDS.sleep(timeUnit);
    } catch (InterruptedException e) {
        e.printStackTrace();
    }
    return "线程池：" + Thread.currentThread().getName() + "paymentInfo_Timeout, id：" + id + "睡眠：" + timeUnit + "秒";
}

public String paymentInfo_TimeoutHandler(Integer id) {
    return "备用响应" + id;
}
```



#### 测试

访问调用了配置了断路器的service方法的controller方法，可以看到在指定时间内没有返回响应，就会返回备选响应。



### 服务消费侧配置

#### 前提准备

一个服务消费者微服务



#### 增加Hystrix依赖

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-hystrix</artifactId>
</dependency>
```



#### 开启断路器功能

在主启动类增加`@EnableHystrix`注解

```java
@SpringBootApplication
@EnableFeignClients
@EnableHystrix
public class OrderHystrixApplication {

    public static void main(String[] args) {
        SpringApplication.run(OrderHystrixApplication.class, args);
    }
}
```



#### 业务配置

在Controller配置Hystrix，并指定备选方法。

```java
@RestController
public class OrderHystrixController {

    @Autowired
    private PaymentHystrixService paymentHystrixService;

    @GetMapping("/consumer/payment/hystrix/ok/{id}")
    public String paymentInfo_OK(@PathVariable Integer id) {
        String result = paymentHystrixService.paymentInfo_OK(id);
        return result;
    }

    @GetMapping("/consumer/payment/hystrix/timeout/{id}")
    @HystrixCommand(fallbackMethod = "paymentTimeOutFallbackMethod", commandProperties = {
            @HystrixProperty(name = "execution.isolation.thread.timeoutInMilliseconds", value = "1500")
    })
    public String paymentInfo_Timeout(@PathVariable Integer id) {
        String result = paymentHystrixService.paymentInfo_Timeout(id);
        return result;
    }

    public String paymentTimeOutFallbackMethod(@PathVariable("id") Integer id) {
        return "消费侧备选响应";
    }
}
```



> 当服务提供侧配置了降级方法，服务消费侧也提供了降级方法。那么返回的服务消费侧提供的降级方法。





### 服务降级优化

#### 当前存在的问题

按照上面的配置，每个需要服务降级的方法都需要创建一个对应的服务降级方法。这会导致**代码的膨胀**。

另外，业务代码和备选方法混杂在一起，会导致代码的**混乱**。



#### 配置全局服务降级

为了解决**代码的膨胀**的问题，我们可以将通用的降级方法抽取出来，配置成全局服务降级方法。



在Controller类上配置`@DefaultProperties`注解，defaultFallback属性指定全局降级方法。

handler方法上增加`@HystrixCommand`注解，没有指定特定的降级方法的，默认使用全局降级方法。

```java
@DefaultProperties(defaultFallback = "paymentGlobalFallbackMethod")
@RestController
public class OrderHystrixController {

    @Autowired
    private PaymentHystrixService paymentHystrixService;

    @GetMapping("/consumer/payment/hystrix/ok/{id}")
    public String paymentInfo_OK(@PathVariable Integer id) {
        String result = paymentHystrixService.paymentInfo_OK(id);
        return result;
    }

    @GetMapping("/consumer/payment/hystrix/timeout/{id}")
    @HystrixCommand
    public String paymentInfo_Timeout(@PathVariable Integer id) {
        String result = paymentHystrixService.paymentInfo_Timeout(id);
        return result;
    }

    public String paymentTimeOutFallbackMethod(@PathVariable("id") Integer id) {
        return "消费侧备选响应";
    }

    public String paymentGlobalFallbackMethod() {
        return "全局降级方法";
    }
}
```



#### Feign服务降级

在服务消费侧，我们可以通过开启Feign Hystrix的功能，从而实现继承Feign Service接口的方法，从而完成降级方法和业务方法的解耦。



##### yml配置

下面这段配置的作用是开启Feign的hystrix功能：

```yml
feign:
  hystrix:
    enabled: true
```



##### 继承Feign Service接口

```java
@Component
public class PaymentHystrixFallbackService implements  PaymentHystrixService{

    @Override
    public String paymentInfo_OK(Integer id) {
        return "OK的降级方法";
    }

    @Override
    public String paymentInfo_Timeout(Integer id) {
        return "Timeout的降级方法";
    }
}
```



##### @FeignClient注解指定降级方法的类

```java
@Component
@FeignClient(value = "cloud-provider-hystrix-payment", fallback = PaymentHystrixFallbackService.class)
public interface PaymentHystrixService {

    @GetMapping("/payment/hystrix/ok/{id}")
    String paymentInfo_OK(@PathVariable("id") Integer id);

    @GetMapping("/payment/hystrix/timeout/{id}")
    String paymentInfo_Timeout(@PathVariable("id") Integer id);
}
```



## Hystrix服务熔断配置

### 前置准备

一个服务提供者微服务



### 依赖

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-hystrix</artifactId>
</dependency>
```



### 配置

在业务做如下配置

```java
package com.xgc.cloud.service;

import cn.hutool.core.util.IdUtil;
import com.netflix.hystrix.contrib.javanica.annotation.HystrixCommand;
import com.netflix.hystrix.contrib.javanica.annotation.HystrixProperty;
import org.springframework.stereotype.Service;
import org.springframework.web.bind.annotation.PathVariable;

import java.util.concurrent.TimeUnit;

@Service
public class PaymentService {

    //服务熔断
    @HystrixCommand(fallbackMethod = "paymentCircuitBreakerFallback", commandProperties = {
            @HystrixProperty(name = "circuitBreaker.enabled", value = "true"),//是否开启断路器
            @HystrixProperty(name = "circuitBreaker.requestVolumeThreshold", value = "10"),//请求次数
            @HystrixProperty(name = "circuitBreaker.sleepWindowInMilliseconds", value = "10000"),//时间窗口期
            @HystrixProperty(name = "circuitBreaker.errorThresholdPercentage", value = "60")//失败率达到多少后跳闸
    })
    public String paymentCircuitBreaker(@PathVariable("id") Integer id) {
        if (id < 0) {
            throw new RuntimeException("******id 不能为负数");
        }
        String serialNumber = IdUtil.simpleUUID();
        return "调用成功，流水号：" + serialNumber;
    }

    public String paymentCircuitBreakerFallback(@PathVariable("id") Integer id) {
        return "id不能为负数，请稍后重试";
    }
}
```



在controller层做如下配置

```java
@GetMapping("/payment/circuit/{id}")
public String paymenyCircuitBreaker(@PathVariable Integer id) {
    String result = paymentService.paymentCircuitBreaker(id);
    log.info("*******result:" + result);
    return result;
}
```



### 测试

访问上面配置了服务熔断的url。

输入id值为负数，在10秒内的10次请求内错误率达到60%。我们再输入id值为正数，就会发现服务调用的还是降级方法，这表示服务停止服务了。过一段时间，再次输入id值的为整数请求，此时微服务就会正常提供服务。



### 熔断类型

熔断打开：请求不再进行调用当前服务，内部设置时钟一般为MTTR(平均故障处理时间)，当打开时长达到所设时钟则进入半熔断状态。

熔断关闭：熔断关闭不会对服务进行熔断

熔断半开：部分请求根据规则调用当前服务，如果请求成功且符合规则则认为当前服务回复正常，关闭熔断



### 断路器在什么情况下开始起作用

```java
    @HystrixCommand(fallbackMethod = "paymentCircuitBreakerFallback", commandProperties = {
            @HystrixProperty(name = "circuitBreaker.enabled", value = "true"),//是否开启断路器
            @HystrixProperty(name = "circuitBreaker.requestVolumeThreshold", value = "10"),//请求次数
            @HystrixProperty(name = "circuitBreaker.sleepWindowInMilliseconds", value = "10000"),//时间窗口期
            @HystrixProperty(name = "circuitBreaker.errorThresholdPercentage", value = "60")//失败率达到多少后跳闸
```

涉及到断路器的三个重要参数：快照窗口期、请求总数阈值、错误百分比阈值

- 快照窗口期：断路器确定是否打开需要统计一些请求和错误请求，而统计的时间范围就是快照窗口期，默认为最近的10秒。
- 请求总数阈值：在快照窗口期内，必须满足请求总数阈值才有资格熔断。默认为20，意味着在10秒内，如果该Hystrix命令的调用次数不足20次的话，是不会进行熔断的。
- 错误百分比阈值：当请求总数在快照窗口期内超过了请求总数阈值，且请求错误百分比达到了错误百分比阈值，那么就会熔断。





