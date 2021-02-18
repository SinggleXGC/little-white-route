# Netty整合到Log4j

1. 增加依赖

   ```xml
   <dependency>
       <groupId>log4j</groupId>
       <artifactId>log4j</artifactId>
       <version>1.2.17</version>
   </dependency>
   <dependency>
       <groupId>org.slf4j</groupId>
       <artifactId>slf4j-api</artifactId>
       <version>1.7.25</version>
   </dependency>
   <dependency>
       <groupId>org.slf4j</groupId>
       <artifactId>slf4j-log4j12</artifactId>
       <version>1.7.25</version>
       <scope>test</scope>
   </dependency>
   <dependency>
       <groupId>org.slf4j</groupId>
       <artifactId>slf4j-simple</artifactId>
       <version>1.7.25</version>
       <scope>test</scope>
   </dependency>
   ```

2. 编写配置文件

   ```properties
   #log4j.rootLogger=DEBUG, stdout
   #log4j.appender.stdout=org.apache.log4j.ConsoleAppender
   #log4j.appender.stdout.layout=org.apache.log4j.PatternLayout
   #log4j.appender.stdout.layout.ConversionPattern=[%p] %C{1} - %m%n
   ```

   