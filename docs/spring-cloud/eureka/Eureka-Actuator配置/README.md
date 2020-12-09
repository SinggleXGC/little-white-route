# Eureka Actuator配置

## 实例名配置

在Eureka配置页，我们可以看到默认的实例名是以`hostname + servicename + port`的方式命名的

![](./img/image-20201210000614241.png)

我们可以自定义微服务的实例名

```yml
eureka:
  instance:
    instance-id: payment8001
```

配置完成之后的效果

![](./img/image-20201210001921794.png)



## 显示实例IP信息提示

如图所示：

![image-20201210002409311](img/image-20201210002409311.png)

为了解决这个问题，我们可以开启实例IP信息提示

```yml
eureka:
  instance:
    instance-id: payment8001
    prefer-ip-address: true
```

配置完成之后，效果如下：

![](./img/image-20201210003411808.png)

