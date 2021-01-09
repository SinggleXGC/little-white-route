# Semaphore

Semaphore，字面意思是信号量。Semaphore可以控制同时访问的线程个数，通过acquire获取许可，如果没有就等待，通过release释放许可。

Semaphore提供了两个构造器

```java
//参数permits表示许可数目，即同时可以允许多少线程进行访问
public Semaphore(int permits) {
    sync = new NonfairSync(permits);
}

//这个多了一个参数fair表示是否是公平的，即等待时间越久的越先获取许可
public Semaphore(int permits, boolean fair) {
    sync = (fair)? new FairSync(permits) : new NonfairSync(permits);
}
```



下面是Semaphore类比较重要的几个方法

```java
public void acquire() throws InterruptedException {  }     //获取一个许可
public void acquire(int permits) throws InterruptedException { }    //获取permits个许可
public void release() { }          //释放一个许可
public void release(int permits) { }    //释放permits个许可
```

acquire()用来获取一个许可，若无许可能够获得，则会一直等待，直到获得许可。

release()用来释放许可。注意，在释放许可之前，必须先获获得许可。

这4个方法都会被阻塞，如果想立即得到执行结果，可以使用下面几个方法：

```java
//尝试获取一个许可，若获取成功，则立即返回true，若获取失败，则立即返回false
public boolean tryAcquire() { };

//尝试获取一个许可，若在指定的时间内获取成功，则立即返回true，否则则立即返回false
public boolean tryAcquire(long timeout, TimeUnit unit) throws InterruptedException { }; 

//尝试获取一个许可，若在指定的时间内获取成功，则立即返回true，否则则立即返回false
public boolean tryAcquire(int permits) { };

//尝试获取permits个许可，若在指定的时间内获取成功，则立即返回true，否则则立即返回false
public boolean tryAcquire(int permits, long timeout, TimeUnit unit) throws InterruptedException { };
```

　另外还可以通过availablePermits()方法得到可用的许可数目。



## 示例

假若一个工厂有5台机器，但是有8个工人，一台机器同时只能被一个工人使用，只有使用完了，其他工人才能继续使用。那么我们就可以通过Semaphore来实现：

```java
public class SemaphoreTest {

  public static void main(String[] args) {

    int n = 8; //8个工人
    Semaphore semaphore = new Semaphore(5); //5台机器

    for (int i=0; i<n; i++) {
      new Worker(i+1, semaphore).start();
    }

  }

  static class Worker extends Thread {
    private int num;
    private Semaphore semaphore;

    public Worker(int num, Semaphore semaphore) {
      this.num = num;
      this.semaphore = semaphore;
    }

    @Override
    public void run() {
      try {
        semaphore.acquire();
      } catch (InterruptedException e) {
        e.printStackTrace();
      }
      System.out.println(String.format("工人%d操作机器", num));
      try {
        Thread.sleep(2000);
      } catch (InterruptedException e) {
        e.printStackTrace();
      }
      System.out.println(String.format("工人%d操作完成---", num));
      semaphore.release();
    }
  }

}
```

运行结果：

```
工人2操作机器
工人5操作机器
工人1操作机器
工人4操作机器
工人3操作机器
工人5操作完成---
工人2操作完成---
工人6操作机器
工人7操作机器
工人1操作完成---
工人4操作完成---
工人3操作完成---
工人8操作机器
工人7操作完成---
工人6操作完成---
工人8操作完成---
```



