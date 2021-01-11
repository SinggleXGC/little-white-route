# ConfigServer搭建使用

## ConfigServer的搭建

### 增加依赖

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-config-server</artifactId>
</dependency>
```



### yml配置

```yml
server:
  port: 3344

spring:
  application:
    name: cloud-config-center
  cloud:
    config:
      server:
        git:
          uri: https://gitee.com/Singgle/springcloud-config.git
          search-paths:
            - springcloud-config
      label: master


eureka:
  client:
    service-url:
      defaultZone: http://localhost:7001/eureka
```



### 创建启动类

```java
@SpringBootApplication
@EnableConfigServer
public class MainAppConigCenter3344 {

    public static void main(String[] args) {
        SpringApplication.run(MainAppConigCenter3344.class, args);
    }
}
```



## ConfigServer的使用

先准备一个项目springcloud-config，项目里面存放配置文件。

config-dev.yml

```yml
config:
  info: "master branch, springcloud-config/config-dev.yml version=1"
```



访问http://127.0.0.1:3344/master/config-dev.yml

在页面上可以获取文件配置信息。

```
config:
  info: master branch, springcloud-config/config-dev.yml version=1
```



## 配置读取规则

访问路径规则有如下三种，其中返回信息不太相同。



- /{label}/{application}-{profile}.yml（推荐）

  ```
  config:
    info: master branch, springcloud-config/config-dev.yml version=1
  ```

- /{application}-{profile}.yml

  默认找master分支

  ```
  config:
    info: master branch, springcloud-config/config-dev.yml version=1
  ```

- /{application}/{profile}[/{label}]

  如果不指定label分支，那么默认找master分支

  ```
  {"name":"config","profiles":["dev.yml"],"label":null,"version":"aef2064b190db2c0015dde9369a07fa3cfea8d0a","state":null,"propertySources":[]}
  ```



## ConfigClient配置使用

### bootstrap.yml是什么

- application.yml是用户级的资源配置项

- bootstrap.xml是系统级的，优先级更高

SpringCloud会创建一个Bootstrap Context，作为Spring应用的Application Context的**父上下文**。初始化的时候，Bootstrap Context负责从外部源加载配置属性并解析配置。这两个上下文共享一个从外部获取的Environment。

Bootstrap属性有高优先级，默认情况下，它们不会被本地配置覆盖。Bootstrap Context和Application Context有着不同的约定，所以新增了一个bootstrap文件，保证Bootstrap Context和Application Context配置的分离。



### ConfigClient的配置

#### 增加依赖

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-config</artifactId>
</dependency>
```



#### 配置

```yaml
server:
  port: 3355

spring:
  application:
    name: config-client
  cloud:
    config:
      label: master
      name: config
      profile: dev
      uri: http://localhost:3344 #配置中心地址
```



### 存在的问题

到这里，ConfigClient启动的时候就能够从配置中心读取配置了。但是问题在于，当配置中心的配置修改之后，ConfigClient不能动态获取更新后的配置。

解决方法下面步骤



#### 增加监控依赖

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```



#### 修改yml，暴露监控端口

在原先yml基础上增加

```yml
management:
  endpoints:
    web:
      exposure:
        include: "*"
```



#### 增加注解@RefreshScope

在Controller加上@RefreshScope注解



#### 发送请求

ConfigClient启动后，且配置文件修改后，向ConfigClient发送post请求，表示配置修改

```
curl -X POST "http://localhost:3355/actuator/refresh"
```

 

#### 最后的问题

这里就可以解决配置更新后，ConfigClient不能自动更新的问题。

但是还是需要发送请求。

当一个微服务存在多个集群，那么每个集群都要发送一次请求。这是很不合理的。所以需要一种方式能够广播。于是就引入了SpringCloud Bus。

