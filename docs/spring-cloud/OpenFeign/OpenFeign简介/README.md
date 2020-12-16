# OpenFeign简介

## 什么是OpenFeign

Feign是一个声明式WebService客户端。使用Feign能让编写WebService客户端更加简单。

它的使用方法是定义一个服务接口然后在上面添加注解。Feign也支持可拔插式的编码器和解码器。SpringCloud对Fegin进行了封装，使其支持了SpringMVC标准注解和HttpMessageConverters。Feign可以与Eureka和Ribbon组合使用以支持负载均衡。



## OpenFeign能干什么

Feign旨在使编写Java Http客户端变得更容易。

之前服务的调用是通过Ribbon+RestTemplate完成的。在实际开发中，**对同一接口的调用可能不止一处，所以通常会针对每一个微服务自行封装一些客户端类来包装对这些依赖服务的调用**。

所以，Feign在此基础上做了进一步封装，由他来帮助我们定义和实现依赖服务接口的定义。**在Feign的实现下，我们只需创建一个接口并使用注解的方式来配置它**，即可完成对服务提供方的接口绑定，简化了使用SpringCloud Ribbon的使用时要封装服务调用客户端的开发量。

> Feign继承了Ribbon
> 利用Ribbon维护了服务提供者的服务列表信息，并使用轮询实现了客户端的负载均衡。而与Ribbon不同的是，**通过Feign只需要定义一个服务绑定接口且以声明式的方法**，优雅而简单的实现了服务调用。



## Feign和OpenFeign的区别

| Feign                                                        | OpenFeign                                                    |      |
| ------------------------------------------------------------ | ------------------------------------------------------------ | ---- |
| Feign是SpringCloud组件中的一个轻量级RESTful的HTTP服务客户端。Feign内置了Ribbon，用来做客户端负载均衡。 | OpenFeign是SpringCloud在Feign的基础上支持SpringMVC注解，如@RequestMapping等等。OpenFeign的@FeignClient可以解析SpringMVC的@RequestMapping注解下的接口，并通过动态代理的方式产生实现类，实现类中做负载均衡并调用其他服务。 |      |

