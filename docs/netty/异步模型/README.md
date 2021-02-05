# 异步模型

## 什么是异步模型

1. Netty的IO操作是异步的，Bind、Write、Connect等操作会简单的返回一个ChannelFuture
2. 调用者不能立即获得结果，而是通过Future-Listener机制，用户可以方便的主动获得或者通过通知机制获得IO操作结果
3. Netty的异步模型是建立在Future和Callback之上的。Callback就是回调。重点说Future，它的核心思想是：假设一个fun方法耗时长，那么可以在调用Fun方法的时候立即返回一个Future对象，后续可以通过Future对象去监控fun方法的处理过程(即Future-Listener机制)



## Future说明

1. 表示异步的执行结果，可以通过它提供的方法来检测执行是否完成，比如检索计算等。

2. ChannelFuture是一个接口，`public interface ChannelFuture extends Future<Void>`

   我们可以添加监听器，当监听的事件发生时，就会通知监听器

   ```java
   ChannelFuture channelFuture = bootstrap.bind(6668).sync();
   
   //给ChannelFuture注册监听器，监控我们关心的事件
   channelFuture.addListener(new ChannelFutureListener() {
       @Override
       public void operationComplete(ChannelFuture channelFuture) throws Exception {
           if (channelFuture.isSuccess()) {
               System.out.println("监听端口6668成功");
           } else {
               System.out.println("监听端口失败");
           }
       }
   });
   ```

   