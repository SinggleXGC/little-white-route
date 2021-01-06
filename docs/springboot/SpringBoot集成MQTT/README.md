# SpringBoot集成MQTT

[springboot与mqtt整合依赖Demo - gitee](https://gitee.com/Singgle/mqttpro)

[导入mqttpro依赖完成mqtt接收方和订阅方的demo](https://gitee.com/Singgle/mqtt-client)

[spring与mqtt整合没有分离出依赖的demo](https://gitee.com/Singgle/mqttx)



## 导入依赖

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-integration</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.integration</groupId>
    <artifactId>spring-integration-stream</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.integration</groupId>
    <artifactId>spring-integration-mqtt</artifactId>
</dependency>
```



## 自定义属性配置

通过@ConfigurationProperties注解使得该类能够与配置文件中的对应属性绑定起来

```java
package com.xgc.mqtt.config;

import lombok.Data;
import org.springframework.boot.context.properties.ConfigurationProperties;
import org.springframework.stereotype.Component;

@Data
@Component
@ConfigurationProperties(prefix = "spring.mqtt")
public class MqttProperties {

  private String username;

  private String password;

  private String hostUrl;

  private int completionTimeout;

  private Client client = new Client();

  private Topic topic = new Topic();

  @Data
  public static class Client {
    private String id;
  }

  @Data
  public static class Topic {
    private String defaultTopic;
    private String myTopic;
  }

}
```



## mqtt配置

```java
package com.xgc.mqtt.config;

import com.xgc.mqtt.handler.MqttReceiveHandler;
import lombok.extern.slf4j.Slf4j;
import org.eclipse.paho.client.mqttv3.MqttConnectOptions;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.integration.annotation.IntegrationComponentScan;
import org.springframework.integration.annotation.ServiceActivator;
import org.springframework.integration.channel.DirectChannel;
import org.springframework.integration.core.MessageProducer;
import org.springframework.integration.mqtt.core.DefaultMqttPahoClientFactory;
import org.springframework.integration.mqtt.core.MqttPahoClientFactory;
import org.springframework.integration.mqtt.inbound.MqttPahoMessageDrivenChannelAdapter;
import org.springframework.integration.mqtt.outbound.MqttPahoMessageHandler;
import org.springframework.integration.mqtt.support.DefaultPahoMessageConverter;
import org.springframework.messaging.MessageChannel;
import org.springframework.messaging.MessageHandler;

@Slf4j
@Configuration
//@IntegrationComponentScan注解配置被@Configuration标注的类，以及为被@MessagingGateway注解标注的接口生成GatewayProxyFactoryBean
@IntegrationComponentScan
public class MqttConfig {

  @Autowired
  private MqttProperties mqttProperties;

  @Autowired
  private MqttReceiveHandler mqttReceiveHandler;

  /**
   * MQTT连接器选项
   */
  @Bean
  public MqttConnectOptions getMqttConnextOptions() {
    MqttConnectOptions mqttConnectOptions = new MqttConnectOptions();
    //clean session表示MQTT客户端向服务器发起CONNECT请求时，是创建持久会话还是临时会话。
    //false表示持久会话，客户断开连接后，会话会保留至超时注销。这样该客户端订阅的主题会保留在EMQ服务器内存上。
    //true表示临时会话，客户端断开连接后，该客户端订阅的主题不会保存在EMQ服务器上
    mqttConnectOptions.setCleanSession(true);
    mqttConnectOptions.setConnectionTimeout(10);
    mqttConnectOptions.setAutomaticReconnect(true);
    mqttConnectOptions.setUserName(mqttProperties.getUsername());
    mqttConnectOptions.setPassword(mqttProperties.getPassword().toCharArray());

    String hostUrl = mqttProperties.getHostUrl();
    String[] urls = parseHostUrl(hostUrl);

    mqttConnectOptions.setServerURIs(urls);
    mqttConnectOptions.setKeepAliveInterval(10);

    return mqttConnectOptions;
  }

  private String[] parseHostUrl(String hostUrls) {
    hostUrls = hostUrls.replaceAll(" ", "");
    String[] urls = hostUrls.split(",");
    return urls;
  }

  /**
   * MQTT工厂
   */
  @Bean
  public MqttPahoClientFactory mqttPahoClientFactory() {
    DefaultMqttPahoClientFactory factory = new DefaultMqttPahoClientFactory();
    factory.setConnectionOptions(getMqttConnextOptions());
    return factory;
  }

  /**
   * MQTT通道信息通道 (生产者)
   */
  @Bean
  public MessageChannel mqttOutboundChannel() {
    return new DirectChannel();
  }

  /**
   * MQTT消息处理器（生产者）
   */
  @Bean
  @ServiceActivator(inputChannel = "mqttOutboundChannel")
  public MessageHandler mqttOutbound() {
    // 在这里进行mqttOutboundChannel的相关设置
    MqttPahoMessageHandler messageHandler =
        new MqttPahoMessageHandler(mqttProperties.getClient().getId(), mqttPahoClientFactory());
    // 如果设置成true，发送消息时将不会阻塞
    messageHandler.setAsync(true);
    messageHandler.setDefaultTopic(mqttProperties.getTopic().getDefaultTopic());
    return messageHandler;
  }

  /**
   * MQTT信息通道(消费者)
   */
  @Bean
  public MessageChannel mqttInputChannel() {
    return new DirectChannel();
  }

  /**
   * 配置client、绑定监听的topic(消费者)
   */
  @Bean
  public MessageProducer inbound() {
    MqttPahoMessageDrivenChannelAdapter adapter =
        new MqttPahoMessageDrivenChannelAdapter(
            mqttProperties.getClient().getId().concat("_inbound1"), mqttPahoClientFactory(),
            mqttProperties.getTopic().getDefaultTopic(), mqttProperties.getTopic().getMyTopic()
        );
    adapter.setCompletionTimeout(mqttProperties.getCompletionTimeout());
    adapter.setConverter(new DefaultPahoMessageConverter());
    adapter.setQos(2);
    adapter.setOutputChannel(mqttInputChannel());
    return adapter;
  }

  /**
   * MQTT消息处理器(消费者)
   */
  @Bean
  @ServiceActivator(inputChannel = "mqttInputChannel")
  public MessageHandler handler() {
    return message ->
        //处理接受信息
        mqttReceiveHandler.handle(message);
  }
  
}

```



## 发布消息handler

生产者发送消息的handler

```java
package com.xgc.mqtt.handler;

import org.springframework.integration.annotation.MessagingGateway;
import org.springframework.integration.mqtt.support.MqttHeaders;
import org.springframework.messaging.handler.annotation.Header;

@MessagingGateway(defaultRequestChannel = "mqttOutboundChannel")
public interface MqttPublisherHandler {

  /**
   * 发送信息到MQTT服务器
   * @param data 发送的文本
   */
  void sendToMqtt(String data);

  /**
   * 发送信息到MQTT服务器
   * @param topic 主题
   * @param payload 消息文本
   */
  void sendToMqtt(@Header(MqttHeaders.TOPIC) String topic, String payload);

  /**
   * 发送信息到MQTT服务器
   * @param topic 主题
   * @param qos 服务质量等级
   * @param payload 消息文本
   */
  void sendToMqtt(
      @Header(MqttHeaders.TOPIC) String topic, @Header(MqttHeaders.QOS) int qos, String payload);

}
```



## 接受消息handler

消费者处理消息的handler

```java
package com.xgc.mqtt.handler;

import org.springframework.messaging.Message;

public interface MqttReceiveHandler {

  void handle(Message<?> message);

}
```



## spring.factories文件配置

该文件所在路径为：resources/META-INF/spring.factories

```factories
org.springframework.boot.autoconfigure.EnableAutoConfiguration=\
  com.xgc.mqtt.config.MqttConfig,\
  com.xgc.mqtt.config.MqttProperties
```



## 测试

完成上面的步骤，之后一个Spring集成了MQTT的依赖就完成了，我们的使用就很简单了。



### 消息消费端构建

#### 导入依赖

导入上述依赖

```xml
<dependency>
    <groupId>com.xgc</groupId>
    <artifactId>mqttpro</artifactId>
    <version>1.0-SNAPSHOT</version>
</dependency>
```



#### 增加mqtt配置

application.yml文件配置

```xml
server:
  port: 9001

spring:
  mqtt:
    username: admin
    #MQTT-密码password: password
    password: public
    #MQTT-服务器连接地址，如果有多个，用逗号隔开，如：tcp://127.0.0.1:1883，tcp://192.168.2.133:1883
    #    url: tcp://192.168.101.146:1883
    hostUrl: tcp://123.57.159.15:1883, tcp://10.0.1.117:1883
    #    url: tcp://127.0.0.1:1883
    #MQTT-连接服务器默认客户端ID
    client:
      id: xgc-client
    #MQTT-默认的消息推送主题，实际可在调用接口时指定
    topic:
      defaultTopic: /voerka/hispro/devices/#
      myTopic: /voerka/hispro/devices/#
    #连接超时
    completionTimeout: 3000
```



#### 实现MqttReceiveHandler接口

```java
@Slf4j
@Component
public class MqttConsumerHandler implements MqttReceiveHandler {

  public void handle(Message<?> message) {
    String payload = message.getPayload().toString();
    String topic = message.getHeaders().get("mqtt_receivedTopic").toString();
    log.info(topic);
    log.info(payload);
  }

}
```



### 消息发送端构建

#### 注入MqttPublisherHandler来构建service层

```java
@Service
public class MqttProducerService {

 @Autowired
  private MqttPublisherHandler mqttPublisherHandler;

  public void sendMessage(String message) {
    mqttPublisherHandler.sendToMqtt("/voerka/hispro/devices/1", message);
  }
}
```



#### 构建controller层提供测试接口

```java
@RestController
public class MqttProducerController {

  @Autowired
  private MqttProducerService mqttProducerService;

  @GetMapping("/producer")
  public String sendMessage() {
    mqttProducerService.sendMessage("producer message");
    return "success";
  }
}
```



### 访问接口进行测试

访问`http://localhost:9001/producer`接口，可以看到控制台输出了接受消息的topic和payload.

```
2021-01-06 09:23:52.428  INFO 13852 --- [client_inbound1] com.xgc.test.handle.MqttConsumerHandler  : /voerka/hispro/devices/1
2021-01-06 09:23:52.428  INFO 13852 --- [client_inbound1] com.xgc.test.handle.MqttConsumerHandler  : producer message
```



