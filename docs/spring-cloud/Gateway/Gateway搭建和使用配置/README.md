# Gateway搭建和使用配置

## Gateway搭建

### 增加依赖

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-gateway</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
</dependency>
```



### Gateway配置

```yaml
server:
  port: 9527

spring:
  application:
    name: cloud-gateway

eureka:
  instance:
    hostname: cloud-gateway-service
  client:
    service-url:
      register-with-eureka: true
      fetch-registry: true
      defaultZone: http://eureka7001.com:7001/eureka, http://eureka7002.com:7002/eureka
```



### 在主启动类增加注解

```java
@SpringBootApplication
@EnableEurekaClient
public class GatewayApplication {

    public static void main(String[] args) {
        SpringApplication.run(GatewayApplication.class, args);
    }
}
```



## Gateway使用配置

### 路由映射配置

路由映射配置有两种方式：yml文件配置和代码配置



#### yml文件配置

```yaml
spring:
  application:
    name: cloud-gateway
  cloud:
    gateway:
      routes:
        - id: payment_route #路由的ID，没有固定规则但是要求唯一。建议配合服务名
          uri: http://localhost:8001 #匹配后提供服务的路由地址
          predicates:
          	#这里会匹配到服务路径是 http://localhost:8001/payment/{id}
            - Path=/payment/** #断言，路由相匹配的进行路由

        - id: payment_route2
          uri: http://localhost:8001
          predicates:
            - Path=/payment/discovery
```



#### 代码配置

下面这段代码的实现效果是：

`https://news.baidu.com/guonei` 是百度国内新闻地址

它将 `http://localhost:9527/guonei` 映射成了 `https://news.baidu.com/guonei`。所以当我们访问 `http://localhost:9527/guonei `的时候，可以访问百度国内新闻的内容。

```java
package com.xgc.cloud.config;

import org.springframework.cloud.gateway.route.RouteLocator;
import org.springframework.cloud.gateway.route.builder.RouteLocatorBuilder;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class GatewayConfig {

    @Bean
    public RouteLocator customRouteLocator(RouteLocatorBuilder routeLocatorBuilder) {
        RouteLocatorBuilder.Builder routes = routeLocatorBuilder.routes();

        routes.route("path_route_xgc", //路由id
                r -> r.path("/guonei") //路径
                        .uri("https://news.baidu.com/guonei") //服务地址
        ).build();
        return routes.build();
    }
}
```



### 动态路由配置

上面的配置都是固定一个程序提供服务的，但是在真实环境中，是通过集群来向外提供服务。

默认情况下，Gateway会根据注册中心注册的服务列表。以注册中心上**微服务名为路径创建动态路由进行转发**,从而实现动态路由。

通过spring.cloud.gateway.discovery.locator.enabled=true开启动态路由功能。同时uri指定的服务地址不再是固定的了，而是`lb://服务名`的形式。

```yml
spring:
  application:
    name: cloud-gateway
  cloud:
    gateway:
      discovery:
        locator:
          enabled: true #开启从注册中心动态创建路由的功能，利用服务名进行路由
      routes:
        - id: payment_route #路由的ID，没有固定规则但是要求唯一。建议配合服务名
          uri: lb://cloud-payment-service #匹配后提供服务的路由地址
          predicates:
            - Path=/payment/** #断言，路由相匹配的进行路由

        - id: payment_route2
          uri: lb://cloud-payment-service
          predicates:
            - Path=/payment/discovery
```



### Predicate介绍

Predicate断言，满足predicate指定的条件就可以实现路由匹配。

SpringCloud Gateway包含许多内置的Route Prediacte工厂。所有这些Predicate都与HTTP请求的不同属性匹配。多个Route Predicate工厂可以组合使用。



#### 常用的Route Predicate

1. After Route Predicate

2. Before Route Predicate

3. Between Route Predicate

4. Cookie Route Predicate

5. Header Route Predicate

6. Host Route Predicate

7. Method Route Predicate

8. Path Route Predicate
9. Query Route Predicate



#### After Route Predicate

这个断言可以指定在指定时间之后启用该路由匹配规则。

```yml
spring:
  cloud:
    gateway:
      routes:
        - id: payment_route
          uri: lb://cloud-payment-service
          predicates:
            - Path=/payment/**
            - After=2021-01-05T00:50:34.814537800+08:00[Asia/Shanghai]
```



该时间串可以通过下面程序来获取

```java
ZonedDateTime zonedDateTime = ZonedDateTime.now();
System.out.println(zonedDateTime);
```



#### Between Route Predicate

表示在指定时间之前有效

```yml
spring:
  cloud:
    gateway:
      routes:
        - id: payment_route
          uri: lb://cloud-payment-service
          predicates:
            - Path=/payment/**
            - Before=2021-01-05T00:50:34.814537800+08:00[Asia/Shanghai]
```



#### Between Route Predicate

表示在指定时间之内有效：

```yml
spring:
  cloud:
    gateway:
      routes:
        - id: payment_route
          uri: lb://cloud-payment-service
          predicates:
            - Path=/payment/**
            - Between=2021-01-05T00:50:34.814537800+08:00[Asia/Shanghai],2021-01-06T00:50:34.814537800+08:00[Asia/Shanghai]
```



#### Cookie Route Predicate

Cookie Route Predicate需要两个参数，一个是Cookie name，一个是正则表达式。

路由规则会通过获取对应的Cookie name和正则表达式去匹配。如果匹配上就会执行路由，如果没有匹配上则不执行。

```xml
spring:
  cloud:
    gateway:
      routes:
        - id: payment_route
          uri: lb://cloud-payment-service
          predicates:
            - Path=/payment/**
            - Cookie=username,xgc
```



#### Header Route Predicate

Header Route Predicate需要两个参数，一个是属性名，另一个是正则表达式。如果匹配就执行路由。

```yaml
spring:
  cloud:
    gateway:
      routes:
        - id: payment_route
          uri: lb://cloud-payment-service
          predicates:
            - Path=/payment/**
            - Header=X-Request-Id, \d+
```



#### Host Route Predicate

Host Route Predicate接收一组参数，一组匹配的域名列表，这个模板是一个ant分割的模板，用.号作为分隔符

它通过参数中的主机地址作为匹配地址。

```yaml
spring:
  cloud:
    gateway:
      routes:
        - id: payment_route
          uri: lb://cloud-payment-service
          predicates:
            - Path=/payment/**
            - Host=**.xgc.com
```



#### Method Route Predicate

```yaml
spring:
  cloud:
    gateway:
      routes:
        - id: payment_route
          uri: lb://cloud-payment-service
          predicates:
            - Path=/payment/**
            - Method=GET
```



#### Path Route Predicate

```yaml
spring:
  cloud:
    gateway:
      routes:
        - id: payment_route
          uri: lb://cloud-payment-service
          predicates:
            - Path=/payment/**
```



#### Query Route Predicate

```yaml
spring:
  cloud:
    gateway:
      routes:
        - id: payment_route
          uri: lb://cloud-payment-service
          predicates:
            - Path=/payment/**
            - Query=username, \d+
```



### Filter过滤

指的是Spring框架中GatewayFilter的实例，通过过滤器可以在请求被路由前后进行处理。



#### 自定义过滤器

Spring Gateway内置了许多中GatewayFilter的实现类。这里介绍如何自定义过滤器。

过滤器要实现`GlobalFilter`和`Ordered`接口。



自定义过滤器示例如下：

```java
/**
 * 实现功能：
 * 判断请求是否存在uname参数，如果不存在则禁止访问
 */
@Component
@Slf4j
public class MyLogGatewayFilter implements GlobalFilter, Ordered {

    @Override
    public Mono<Void> filter(ServerWebExchange exchange, GatewayFilterChain chain) {
        log.info("request into MyLogGatewayFilter");

        String uname = exchange.getRequest().getQueryParams().getFirst("uname");
        if (uname == null) {
            log.info("uname is null, please login first");
            exchange.getResponse().setStatusCode(HttpStatus.NOT_ACCEPTABLE);
            return exchange.getResponse().setComplete();
        }

        return chain.filter(exchange);
    }

    @Override
    public int getOrder() {
        return 0;
    }
}
```



