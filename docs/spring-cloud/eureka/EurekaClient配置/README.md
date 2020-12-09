# EurekaClient配置

## EurekaClient发送心跳时间

```yml
eureka:
  client:
    #EurekaClient客户端向服务器发送心跳的时间间隔，单位为秒(默认30秒)
    lease-renewal-interval-in-seconds: 1
    #EurekaServer在收到最后一次心跳后等待上限，超出这个时间就会注销服务
    lease-expiration-duration-in-seconds: 2
```



## EurekaClient没有发送心跳多久后注销服务

```yml
eureka:
  client:
    #EurekaServer在收到最后一次心跳后等待上限，超出这个时间就会注销服务
    lease-expiration-duration-in-seconds: 2
```

