# CountDownLatch

CountDownLatch类位于java.util.concurrent包下，利用它可以实现类似计数器的功能。比如有一个任务A，它需要等待其他4个任务完成，才能执行。这种情况，就可以使用CountDownLatch来完成这个功能。



CountDownLatch提供了一个构造器

```java
public CountDownLatch(int count) { }; //参数count为计数值
```

然后下面这3个方法是CountDownLatch类中最重要的方法：

```java
//调用await方法的线程会被挂起，直至count值为0才继续
public void await() throws InterruptedException {}; 

//和await类似，不过等待指定时间count还没有变成0就会继续执行
public boolean await(long timeout, TimeUnit unit) throws InterruptedException {};

//将count值减1
public void countDown() {}; //将count值减1
```



下面是CountDownLatch使用示例

实现效果是，主线程等待两个子线程执行完成才能继续执行。

```java
public class CountDownLatchTest {

  public static void main(String[] args) {
    final CountDownLatch countDownLatch = new CountDownLatch(2);

    for (int i=0; i<2; i++) {
      new Thread() {
        @Override
        public void run() {
          System.out.println(String.format("子线程%s正在执行", Thread.currentThread().getName()));
          try {
            Thread.sleep(5000);
          } catch (InterruptedException e) {
            e.printStackTrace();
          }
          countDownLatch.countDown();
          System.out.println(String.format("子线程%S执行完成", Thread.currentThread().getName()));
        }
      }.start();
    }

    System.out.println("等待所有子线程执行完成");
    try {
      countDownLatch.await();
    } catch (InterruptedException e) {
      e.printStackTrace();
    }
    System.out.println("所有子线程执行完成");
  }

}
```

执行结果为：

```
等待所有子线程执行完成
子线程Thread-1正在执行
子线程Thread-0正在执行
所有子线程执行完成
子线程THREAD-1执行完成
子线程THREAD-0执行完成
```



