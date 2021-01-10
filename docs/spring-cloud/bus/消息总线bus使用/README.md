# 消息总线bus使用

## 设计思想

1. 利用消息总线触发一个客户端/bus/refresh，而刷新其他客户端
2. 利用消息总线触发一个服务器端ConfigServer的/bus/refresh端点，从而触发所有客户端的配置

第二种架构比较好，理由如下：

1. 打破了微服务的职责单一性，因为微服务本身是业务模块，它本不应该承担配置刷新的职责
2. 破坏了微服务各节点的对等性
3. 有一定的局限性。例如，微服务在迁移时，它的网络地址常常会变化，如果想要做到自动刷新，会增加更多的修改



## 全局广播

### 给ConfigCenter增加消息总线支持

#### 增加依赖

```xml
<dependency>
	<groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-bus-amqp</artifactId>
</dependency>
```



#### yml配置

```yml
spring:
    rabbitmq:
        host: 123.57.159.15
        port: 5672
        username: xgc
        password: 123456
	
# rabbitmq相关配置，暴露bus刷新配置的端口
management:
	endpoints:
		web:
			exposure:
				include: 'bus-refresh'
```



### 给ConfigClient增加消息总线支持

#### 增加依赖

```xml
<!-- 增加消息总线rabbitmq支持 -->
<dependency>
	<groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-bus-amqp</artifactId>
</dependency>
```



#### yml配置

```yml
spring:
    rabbitmq:
        host: 123.57.159.15
        port: 5672
        username: xgc
        password: 123456
```



### 测试

项目配置修改之后，发送POST请求

```
curl -X POST “http://localhost:3344/actuator/bus-refresh"
```

这样，多个使用相同配置的ConfigClient就会自动刷新配置了。



## 定点通知

```
curl -X POST “http://localhost:3344/actuator/bus-refresh/{destination}"
curl -X POST "http://localhost:3344/actuator/bus-refresh/config-client:3355"
```

destination = spring.application.name + port