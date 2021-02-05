# Netty简介

## 原生NIO存在的问题

1. NIO的类库和API繁杂，使用麻烦。需要熟悉掌握Selector、ServerSocketChannel、SocketChannel、ByteBuffer等
2. 需要具备其他的额外技能：要熟悉Java多线程编程，因为NIO编程需要涉及到Reactor模式，你必须对多线程和网络编程非常熟悉，才能编写出高质量的NIO程序
3. 开发工作量和难度都非常大：例如客户端面临断连重连、网络闪断、半包读写、失败缓存、网络拥堵和异常流的处理等等
4. JDK NIO的bug：例如臭名昭著的Epoll Bug，它会导致Selector空轮询，最终导致CPU 100%，直至JDK 1.7版本该问题仍然存在，没有被根本解决



## 什么是Netty

1. Netty是JBOSS提供的一个Java开源框架。Netty提供异步的、基于事件驱动的网络应用程序框架，用于快速开发高性能、高可靠性的网络IO程序
2. Netty主要针对在TCP协议下，面向Clients端的高并发应用，或者在Peer-to-Peer场景下的大量数据持续传输的应用
3. Netty是目前最流行的NIO框架，Netty在互联网领域、大数据分布式计算领域、游戏行业、通信行业等获得了广泛的应用，例如：知名的ElasticSearch、Dubbo框架内部都采用了Netty.



## Netty的应用场景

### 互联网行业

在分布式系统中，各个节点之间需要远程服务调用，高性能的RPC框架必不可少，Netty作为异步高性能的通信框架，往往作为基础通信组件被这些RPC框架使用。

典型的应用有：阿里分布式服务框架Dubbo的PRC框架使用Dubbo协议进行节点间通信，Dubbo协议默认使用Netty作为基础通信组件，用于实现各进程节点之间的内部通信。



### 游戏行业

Netty作为高性能的基础通信组件，提供了TCP/UDP和HTTP协议栈，方便定制和开发私有协议栈，账号登陆服务器。

地图服务器之间可以方便的通过Netty进行高性能的通信



### 大数据领域

经典的Hadoop的高性能通信和序列化组件(AVRO 实现数据文件共享)的PRC框架，默认采用Netty进行跨界点通信。

它的Netty Service基于Netty框架二次封装实现。





## Netty的优点

Netty对JDK自带的NIO的API进行了封装，解决了上述问题。

1. 设计优雅。适用于各种传输类型的统一API，阻塞和非阻塞Socket；基于灵活且可扩展的事件模型，可以清晰地分离关注点；高度可定制的线程模型：单线程、一个或多个线程池
2. 使用方便。详细文档，没有其他依赖项，JDK5(Netty 3.x)或6(Netty 4.x)就足够了
3. 高性能、高吞吐量：延迟更低。减少资源消耗。最小化不必要的内存复制
4. 安全。完整的SSL/TSL和StartTLS支持
5. 社区活跃



## Netty版本说明

1. Netty版本分为netty3.x和Netty4.x、Netty5.x
2. 因为Netty5出现重大bug，现在已经被官网废弃了。目前推荐使用的是Netty4.x的稳定版本