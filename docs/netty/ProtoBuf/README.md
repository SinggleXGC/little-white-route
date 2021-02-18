# ProtoBuf

## ProtoBuf简介

1. ProtoBuf是Google发布的开源项目，全称Google Protocol Buffers，是一种轻便高效的结构化数据存储格式，可以用于结构化数据串行化，或者说序列化。它很适合做数据存储或PRC数据交换格式
2. Protobuf是以message的方式来管理数据的
3. 支持跨平台、跨语言
4. 高性能，高可靠性
5. 使用protobuf编译器能自动生成代码，protobuf是将类的定义使用.proto文件进行描述。然后通过protoc.exe编译器根据.proto自动生成.java文件



## ProtoBuf使用

1. 引入依赖

   ```xml
   <dependency>
       <groupId>com.google.protobuf</groupId>
       <artifactId>protobuf-java</artifactId>
       <version>3.6.1</version>
   </dependency>
   ```

2. 编写proto文件

   ```protobuf
   syntax = "proto3"; //版本
   option java_outer_classname = "StudentPOJO"; //生成的外部类名，同时也是文件名
   //protobuf 使用message管理数据
   message Student { //会在StudentPOJO外部类生成一个内部类Student，它是正在发送的POJO对象
       int32 id = 1; //Student类中有一个属性 名字为id，类型为int32(protobuf类型) 1表示属性序号，不是值
       string name = 2;
   }
   ```

3. 编译proto文件为java文件

   下载protoc.exe编译器，执行命令

   ```
   protoc.exe --java_out=. Student.proto
   ```

   就会生成StudentPOJO.java文件

4. 为服务器端增加ProtobufDecoder解码器，为客户端增加ProtobufEncoder编码器

   服务器端需要指定传输的类型

   ```java
   pipeline.addLast("decoder",
                    new ProtobufDecoder(StudentPOJO.Student.getDefaultInstance()));
   ```

   客户端构建Student对象，并发送给服务器端

   ```java
   StudentPOJO.Student student = StudentPOJO.Student.newBuilder().setId(4)
       .setName("豹子头 林冲").build();
   ctx.writeAndFlush(student);
   ```

