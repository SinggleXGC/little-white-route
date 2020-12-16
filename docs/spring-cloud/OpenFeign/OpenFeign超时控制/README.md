# OpenFeign超时控制

OpenFeign默认等待1秒钟，超时就会报错。

但是会存在有时业务处理过长，执行时间超出1秒钟。但是，我们不希望它报错，能够将超时时间延长。

可以在application.yml文件配置如下：

```yml
#设置feign客户端超时时间
ribbon:
  # 建立连接所需时间
  ConnectTimeOut: 5000
  # 建立连接后从服务器读取到可用资源所用的时间
  ReadTimeOut: 5000
```

