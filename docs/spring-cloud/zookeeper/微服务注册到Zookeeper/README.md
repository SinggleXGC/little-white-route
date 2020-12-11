# 微服务注册到Zookeeper

一个微服务要注册到Zookeeper要做如下配置



## 前提准备

1. 启动Zookeeper服务器



## 增加依赖

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-zookeeper-discovery</artifactId>
    <exclusions>
        <exclusion>
            <groupId>org.apache.zookeeper</groupId>
            <artifactId>zookeeper</artifactId>
        </exclusion>
    </exclusions>
</dependency>
<dependency>
    <groupId>org.apache.zookeeper</groupId>
    <artifactId>zookeeper</artifactId>
    <version>3.6.2</version>
    <exclusions>
        <exclusion>
            <groupId>org.slf4j</groupId>
            <artifactId>slf4j-log4j12</artifactId>
        </exclusion>
    </exclusions>
</dependency>
```



## 项目配置

```yml
server:
  port: 8004

spring:
  application:
    name: cloud-provider-payment
  cloud:
    zookeeper:
      #zookeeper服务的地址
      connect-string: localhost:2181
```



## 主启动类增加`@EnableDiscoveryClient`注解

```java
@EnableDiscoveryClient //该注解用于向consul或者zookeeper作为注册中心时注册服务
@SpringBootApplication
public class PaymentMain8004 {

    public static void main(String[] args) {
        SpringApplication.run(PaymentMain8004.class, args);
    }
}
```



## 启动微服务

完成上面的配置之后，我们启动微服务之后，该微服务就会自动注册到Zookeeper中。

验证如下：

```
[root@xgc bin]# ./zkCli.sh 
Connecting to localhost:2181
.....
WATCHER::

WatchedEvent state:SyncConnected type:None path:null
[zk: localhost:2181(CONNECTED) 0] ls /
[services, zookeeper]
[zk: localhost:2181(CONNECTED) 1] ls /services 
[cloud-provider-payment]
[zk: localhost:2181(CONNECTED) 2] ls /services/cloud-provider-payment/4d92fc3a-d87d-46a1-b110-b755a8c4e1c6 
[]
[zk: localhost:2181(CONNECTED) 3] get /services/cloud-provider-payment /
services    zookeeper   
[zk: localhost:2181(CONNECTED) 3] get /services/cloud-provider-payment/4d92fc3a-d87d-46a1-b110-b755a8c4e1c6 
{"name":"cloud-provider-payment","id":"4d92fc3a-d87d-46a1-b110-b755a8c4e1c6","address":"DESKTOP-UCDD0KK","port":8004,"sslPort":null,"payload":{"@class":"org.springframework.cloud.zookeeper.discovery.ZookeeperInstance","id":"application-1","name":"cloud-provider-payment","metadata":{}},"registrationTimeUTC":1607711384382,"serviceType":"DYNAMIC","uriSpec":{"parts":[{"value":"scheme","variable":true},{"value":"://","variable":false},{"value":"address","variable":true},{"value":":","variable":false},{"value":"port","variable":true}]}}
[zk: localhost:2181(CONNECTED) 4] 
```

可以看到微服务的信息就注册到Zookeeper中了。