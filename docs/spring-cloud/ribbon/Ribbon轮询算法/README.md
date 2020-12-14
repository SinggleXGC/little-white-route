# Ribbon轮询算法

## Ribbon轮询算法的原理

rest接口第几次请求数 % 服务器集群总数量 = 实际调用服务器位置下标

每次服务重启动之后，rest接口计数从1开始。



## 源码分析

Ribbon轮询算法对应的类是`RoundRobinRule`，其中choose方法实现了获取对应服务实例的功能。

```java
public Server choose(ILoadBalancer lb, Object key) {}
```

这个方法接收一个ILoadBalancer对象和key作为参数。其中ILoadBalancer对象维护了存储所有服务实例的列表和存储可用服务实例的列表。

`RoundRobinRule`类有一个属性`AtomicInteger`，代表接口被调用的次数。

```java
private AtomicInteger nextServerCyclicCounter;
```



### choose方法实现

choose方法具体实现如下：

```java
//获取具体服务实例的方法
public Server choose(ILoadBalancer lb, Object key) {
        if (lb == null) {
            log.warn("no load balancer");
            return null;
        }

    	//存储要返回的服务实例对象
        Server server = null;
    	//存储获取服务实例的次数
        int count = 0;
        while (server == null && count++ < 10) {
            //获取可用服务列表
            List<Server> reachableServers = lb.getReachableServers();
            //获取所有服务实例列表
            List<Server> allServers = lb.getAllServers();
            int upCount = reachableServers.size();
            int serverCount = allServers.size();

            if ((upCount == 0) || (serverCount == 0)) {
                log.warn("No up servers available from load balancer: " + lb);
                return null;
            }

            //获取要获取的服务实例的下标
            int nextServerIndex = incrementAndGetModulo(serverCount);
            server = allServers.get(nextServerIndex);

            if (server == null) {
                /* Transient. */
                Thread.yield();
                continue;
            }

            if (server.isAlive() && (server.isReadyToServe())) {
                return (server);
            }

            // Next.
            server = null;
        }

        if (count >= 10) {
            log.warn("No available alive servers after 10 tries from load balancer: "
                    + lb);
        }
        return server;
    }
```



### incrementAndGetModulo方法实现

```java
//获取要获取的服务实例的下标
private int incrementAndGetModulo(int modulo) {
    for (;;) {
        int current = nextServerCyclicCounter.get();
        int next = (current + 1) % modulo;
        if (nextServerCyclicCounter.compareAndSet(current, next))
            return next;
    }
}
```



