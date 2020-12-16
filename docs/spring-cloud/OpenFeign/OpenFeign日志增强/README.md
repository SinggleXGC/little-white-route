# OpenFeign日志增强

Feign提供了日志打印的功能，我们可以通过配置来调整日志级别，从而了解Feign中Http请求的细节。



## 日志级别

- NONE：默认的，不显示任何日志
- BASIC：仅记录请求方法，URL，响应状态码和执行时间
- HEADERS：除了BASIC中的内容，还有请求和响应的头信息
- FULL：除了HEADERSd的内容，还有请求和响应的正文及元数据



## OpenFeign开启日志

### 配置日志级别

```java
package com.xgc.cloud.config;

import feign.Logger;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class FeignLogConfig {

    @Bean
    Logger.Level feignLoggerLevel() {
        return Logger.Level.FULL;
    }
}
```



### 配置记录接口

```yml
logging:
  config:
    # feign日志以什么级别监控哪个接口
    com.xgc.cloud.controller.PaymentController: debug
```

