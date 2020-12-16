# @ConfigurationProperties

在编写项目代码的时候，为了能够更灵活的配置。在SpringBoot项目中，我们会将大量的参数配置在`application.properties`或`applicaton.yml`文件中，通过`@ConfigurationProperties`注解，从而获得这些参数。



## @ConfigurationProperties使用

java代码：下面代码自定义了一个enabled参数

```java
@Data
@ConfigurationProperties("swagger")
public class SwaggerProperties {

  private Boolean enabled;

}
```



配置文件：与上面的enabled属性对应的参数写法

```yml
swagger:
	enabled: true
```



仅仅是上面的配置，程序是不能够获取到我们在配置文件写好的参数值。我们需要将`SwaggerProperties`加入到Spring应用上下文。为此，我们在`SwaggerProperties`类上面标注`@Configuration`或`@Component`。

```java
@Data
@Component
@ConfigurationProperties("swagger")
public class SwaggerProperties {

  private Boolean enabled;

}
```



## Spring宽松绑定规则(relaxed binding)

> Spring使用一些宽松的绑定属性规则。因此，以下变体全部绑定到hostName属性上

```properties
mail.hostName=localhost
mail.hostname=localhost
mail.host_name=localhost
mail.host-name=localhost
mail.HOST_NAME=localhost
```



## 启动时检验@ConfigurationProperties

如果我们配置参数在传入到应用中时是有效的，我们可以在字段上`bean validation`注解，同时在类上添加`@Validated`注解。

如下，当我们的enabled为空的时候，应用启动时，会抛出`BindValidationException`。

```java
@Data
@Component
@Validated
@ConfigurationProperties("swagger")
public class SwaggerProperties {

  @NotNull private Boolean enabled;

}
```

通过这种方式，我们也可以自定义自己的校验规则



## 参考资料

[@ConfigurationProperties 注解使用姿势，这一篇就够了](https://www.cnblogs.com/FraserYu/p/11261916.html)

