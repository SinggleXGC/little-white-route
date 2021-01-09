# CyclicBarrier

## 什么是CyclicBarrier

字面意思：回环栅栏。通过它可以实现让一组线程等待至某个状态之后再全部执行。调用await方法，线程就会处于barrier状态。

叫做回环的原因是，当所有等待线程被释放之后，CyclicBarrier可以被重用。



## CyclicBarrier详解

CyclicBarrier类是juc包的，提供两个构造器：

```java
public CyclicBarrier(int parties, Runnable barrierAction) {
}

public CyclicBarrier(int parties) {
}
```

参数parties的意思是让多少个线程处于barrier状态，参数battierAction是当处于barrier状态的线程达到指定个数，就会选取一个线程来执行的任务



CyclicBarrier最重要的方法就是await

```java
public int await() throws InterruptedException, BrokenBarrierException { };

public int await(long timeout, TimeUnit unit)throws InterruptedException,BrokenBarrierException,TimeoutException { };
```

第一个await的意思是让调用这个方法的线程处于barrier状态

第二个await的意思是让调用这个方法的线程处于barrier状态，到指定时间，还有线程没有达到指定状态，就让当前线程继续执行。



## 示例

实现3个线程都写入完成之后，所有线程才能继续执行。

```java
public class CyclicBarrierTest {

  public static void main(String[] args) {

    int n = 3;
    CyclicBarrier cyclicBarrier = new CyclicBarrier(n);

    for(int i= 0 ; i<n; i++) {
      new Writer(cyclicBarrier).start();
    }


  }

  static class Writer extends Thread {

    private final CyclicBarrier cyclicBarrier;

    public Writer(CyclicBarrier cyclicBarrier) {
      this.cyclicBarrier = cyclicBarrier;
    }

    @Override
    public void run() {
      System.out.println(String.format("线程%s正在写入数据", Thread.currentThread().getName()));
      try {
        Thread.sleep(5000); //模拟写入耗时
      } catch (InterruptedException e) {
        e.printStackTrace();
      }
      System.out.println(String.format("线程%s写入数据完成，等待其他线程写入完毕", Thread.currentThread().getName()));

      try {
        cyclicBarrier.await();
      } catch (InterruptedException e) {
        e.printStackTrace();
      } catch (BrokenBarrierException e) {
        e.printStackTrace();
      }

      System.out.println("所有线程写入完成");
    }
  }
}
```

执行结果

```
线程Thread-1正在写入数据
线程Thread-2正在写入数据
线程Thread-0正在写入数据
线程Thread-1写入数据完成，等待其他线程写入完毕
线程Thread-0写入数据完成，等待其他线程写入完毕
线程Thread-2写入数据完成，等待其他线程写入完毕
所有线程写入完成
所有线程写入完成
所有线程写入完成
```



为CyclicBarrier传入一个Runnable任务，这样当所有线程都到达barrier状态之后，就会**选取一个线程**就会执行Runnable任务。

