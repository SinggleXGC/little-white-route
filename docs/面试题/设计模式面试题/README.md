# 设计模式面试题

## 单例模式

### Q1

单例模式的编写：

**饿汉式**

直接实例化

```java
public class Singleton {
    private static final Singleton INSTANCE = new Singleton();
    
    private Singleton() {}
    
    public static Singleton getInstance() {
        return INSTANCE;
    }
}
```



枚举

```java
public enum Singleton {
  INSTANCE;

  public Singleton getInstance() {
    return INSTANCE;
  }
}
```



静态代码块(适合初始化复杂对象)

读取resources文件下的`singleton.properties`文件

```properties
info=xgc
```

代码

```java
public class Singleton {

  public static final Singleton INSTANCE;
  private String info;

  private Singleton(String info) {
    this.info = info;
  }

  static {
    Properties properties = new Properties();
    InputStream inputStream = Singleton.class.getClassLoader().getResourceAsStream("singleton.properties");
    try {
      properties.load(inputStream);
    } catch (IOException e) {
      e.printStackTrace();
    }
    String info = properties.getProperty("info");
    INSTANCE = new Singleton(info);
  }

  public static Singleton getInstance() {
    return INSTANCE;
  }

}
```





**懒汉式**

线程安全写法

```java
public class Singleton {
    private static volatile Singleton INSTANCE;
    
    private Singleton() {}
    
    public static Singleton getInstance() {
        if(INSTANCE == null) {
            synchronized(Singleton.class) {
                if(INSTANCE == null) {
                    INSTANCE = new Singleton();
                }
            }
        }
        return INSTANCE;
    }
}
```



静态内部类

```java
/**
 * 内部类只有被加载和初始化时，才创建INSTANCE
 * 静态内部类不会随着外部类的加载而初始化，它是要单独去加载和初始化的
 * 线程安全
 */
public class Singleton {
  private Singleton() {}

  private static class Inner {
    private static final Singleton INSTANCE = new Singleton();
  }

  public static Singleton getInstance() {
    return Inner.INSTANCE;
  }
}
```

