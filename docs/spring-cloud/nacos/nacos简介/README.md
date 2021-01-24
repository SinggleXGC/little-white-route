# nacos简介

nacos，前四个字母分别为Naming和Configuration的前两个字母，最后的s为service



## 什么是nacos

nacos：Dynamic Naming and Configuration Service

一个更易于构建云原生应用的动态服务发现、配置管理和服务管理平台。

Nacos就是注册中心和配置中心的组合，等价于zookeeper+config+bus。



## 依赖

所有使用nacos的功能需要增加

```xml
<!--spring cloud alibaba 2.1.0.RELEASE-->
<dependency>
    <groupId>com.alibaba.cloud</groupId>
    <artifactId>spring-cloud-alibaba-dependencies</artifactId>
    <version>2.1.0.RELEASE</version>
    <type>pom</type>
    <scope>import</scope>
</dependency>
```

