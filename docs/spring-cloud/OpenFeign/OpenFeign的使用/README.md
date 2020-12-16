# OpenFeign的使用

## 前提准备

一个微服务



## 增加依赖

```xml
<!--openfeign-->
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-openfeign</artifactId>
</dependency>
```



## 在主启动类增加`@EnableFeignClients`注解

```java
@SpringBootApplication
@EnableFeignClients
public class OpenFeignMain80 {

    public static void main(String[] args) {
        SpringApplication.run(OpenFeignMain80.class, args);
    }

}
```



## 创建一个服务接口

创建一个服务**接口**，里面定义的方法是我们服务提供方的接口。通过在该服务接口上标注`@EnableClient`注解可以实现与服务提供方的对应接口绑定，其中value参数指定的是服务提供方的名称。

要注意的是，在服务接口上需要明确指定接口参数名称，如下`@PathVariable("id") Long id`不能写成`@PathVariable Long id`，否则会抛出异常。

```java
@Component
@FeignClient(value = "CLOUD-PAYMENT-SERVICE")
public interface PaymentService {

    @GetMapping("/payment/{id}")
    CommonResult getPayment(@PathVariable("id") Long id);
}
```



## 服务接口的调用

通过在服务接口上面标注``@EnableClient``，我们可以直接调用方法的方式调用服务提供者的接口。

```java
@RestController
public class PaymentController {

    @Autowired
    private PaymentService paymentService;

    @GetMapping("/consumer/payment/{id}")
    public CommonResult getPayment(@PathVariable Long id) {
        return paymentService.getPayment(id);
    }
}
```

可以看到，直接调用对应服务接口的对应方法，即可完成对服务提供者接口的调用。



## 测试

在浏览器输入`http://localhost:80/consumer/payment/1`就可以完成接口的调用。