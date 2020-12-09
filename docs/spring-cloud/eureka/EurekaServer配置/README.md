# EurekaServer配置

## EurekaServer清理无效节点的时间间隔

```yml
eureka:
  server:
    #eureka server清理无效节点的时间间隔，单位毫秒，默认60000毫秒，即60秒
    eviction-interval-timer-in-ms: 2000
```

