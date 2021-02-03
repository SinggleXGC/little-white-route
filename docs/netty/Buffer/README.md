# Buffer

## 基本介绍

缓冲区本质上是一个可以读写数据的内存块，可以理解成是一个**容器对象(含数组)**，该对象提供了**一组方法**，可以更轻松地使用内存块，缓冲区对象内置了一些机制，能够追踪和记录缓冲区的状态变化情况。Channel提供从文件、网络读取数据的渠道，但是读取或写入的数据都必须经由Buffer。



在NIO中，Buffer是一个顶层父类，它是一个抽象类。

常用子类如下：

- ByteBuffer，存储字节数据到缓冲区
- ShortBuffer，存储字符串数据到缓冲区
- CharBuffer，存储字符数据到缓冲区
- IntBuffer，存储整数数据到缓冲区
- LongBuffer，存储长整数数据到缓冲区
- FloatBuffer，存储小数到缓冲区
- DoubleBuffer，存储小数到缓冲区



Buffer类定义了所有的缓冲区都具有的四个属性来提供关于其所包含的数据元素的信息

| 属性     | 描述                                                         |
| -------- | ------------------------------------------------------------ |
| Capacity | 容量，即可以容纳的最大数据量；在缓冲区创建时被设定并且不能改变 |
| Limit    | 表示缓冲区的当前终点，不能对缓冲区超出极限的位置进行读写操作。且极限是可以修改的 |
| Position | 位置，下一个要被读或写的元素的索引，每次读写缓冲区数据时都会改变此值，为下次读写做准备 |
| Mark     | 标记，调用mark()来设置mark = position，再调用reset()可以让position恢复到标记的位置 |



## 基本使用

```java
public class BasicBuffer {

    //举例说明Buffer的使用(说明)
    public static void main(String[] args) {
        //创建一个Buffer，大小为5，即可以存放5个int
        IntBuffer intBuffer = IntBuffer.allocate(5);

        //向Buffer存放数据
        for (int i = 0; i < intBuffer.capacity(); i++) {
            intBuffer.put(i*2);
        }

        //如何从buffer读取数据
        //将buffer切换，读写切换
        intBuffer.flip();

        while (intBuffer.hasRemaining()) {
            System.out.println(intBuffer.get());
        }
    }
}
```

输出结果

```
0
2
4
6
8
```



## Buffer常用方法

```java
public abstract class Buffer {
    //JDK1.4时，引入的api
    public final int capacity() //返回此缓冲区的容量
    public final int position() //返回此缓冲区的位置
    public final Buffer position(int newPosition) //设置此缓冲区的位置
    public final limit() //返回此缓冲区的限制
    public final Buffer limit(int newLimit) //设置此缓冲区的限制
    public final Buffer mark() //在此缓冲区的位置设置标记
    public final Buffer reset() //将此缓冲区的位置重置为之前标记的位置
    public final Buffer clear() //清除此缓冲区，即将各个标记恢复到初始状态，但是数据并没有真正擦除
    public final Buffer flip() //反转此缓冲区
    public final Buffer rewind() //重绕此缓冲区
    public final int remaining() //返回当前位置与限制之间的元素值
    public final boolean hasRemaining() //在当前位置与限制之间是否有元素
    public abstract boolean isReadOnly(); //此缓冲区是否为只读缓冲区
    
    //JDK1.6时引入的api
    public abstract boolean hasArray(); //此缓冲区是否具有可访问的底层实现数组
    public abstract Object array(); //返回此缓冲区的底层实现数组
    public abstract int arrayOffset(); //返回此缓冲区的底层实现数组中第一个缓冲区元素的偏移量
    public abstract boolean isDirect(); //告知此缓冲区是否为直接缓冲区
}
```

​	

### ByteBuffer常用方法

```java
public abstract class ByteBuffer {
    //缓冲区创建相关api
    public static ByteBuffer allocateDirect(int capacity) //创建直接缓冲区
    public static ByteBuffer allocate(int capacity) //设置缓冲区的初始容量
    public static ByteBuffer wrap(byte[] array) //把一个数组放到缓冲区中使用
    //构建初始化位置offset和上界length的缓冲区
    public static ByteBuffer wrap(byte[] array, int offset, int length)
    
    //缓存区存取相关api
    public abstract byte get(); //从当前位置position获取，获取之前position+1
    public abstract byte get(int index); //从绝对位置get
    public abstract ByteBuffer put(byte b); //从当前位置增加，position自动+1
    public abstract ByteBuffer put(int index, byte b); //从绝对位置put
}
```



## MappedByteBuffer

NIO提供了MappedByteBuffer，可以让文件直接在内存(堆外的内存)中进行修改，而如何同步到文件由NIO来完成。

```java
public class MappedByteBufferTest {
    public static void main(String[] args) throws IOException {
        RandomAccessFile randomAccessFile =
                new RandomAccessFile("1.txt", "rw");
        FileChannel fileChannel = randomAccessFile.getChannel();

        /**
         * 参数1：读写模式
         * 参数2：可以直接修改的起始位置
         * 参数3：文件映射到内存的大小，即1.txt的5个字节可以映射到内存
         */
        MappedByteBuffer mappedByteBuffer = fileChannel.map(FileChannel.MapMode.READ_WRITE, 0, 5);
        mappedByteBuffer.put(1, (byte) 'a');
        mappedByteBuffer.put(2, (byte) '9');

        randomAccessFile.close();
    }
}
```



## Buffer的分散和聚集

```java
/**
 * Scattering：将数据写入到buffer中。可以采用buffer数组，依次写入 [分散]
 * Gathering：从buffer读取数据时，可以采用Buffer数组，依次读
 */
public class ScatteringAndGatheringTest {

    public static void main(String[] args) throws IOException {
        //使用ServerSocketChannel 和 SocketChannel 网络
        ServerSocketChannel serverSocketChannel = ServerSocketChannel.open();
        InetSocketAddress inetSocketAddress = new InetSocketAddress(7000);

        //绑定端口到socket，并启动
        serverSocketChannel.socket().bind(inetSocketAddress);

        //创建buffer数组
        ByteBuffer[] byteBuffers = new ByteBuffer[2];
        byteBuffers[0] = ByteBuffer.allocate(5);
        byteBuffers[1] = ByteBuffer.allocate(3);

        //等待客户端连接(telnet)
        SocketChannel socketChannel = serverSocketChannel.accept();
        int messageLength = 8; //假定从客户端接收8个字节

        //循环的读取
        while(true) {
            int byteRead = 0;

            while (byteRead < messageLength) {
                long l = socketChannel.read(byteBuffers);
                byteRead += 1; //累计读取的字节数
                System.out.println("byteRead" + byteRead);
                //使用流打印，看看当前的这个buffer的position和limit
                Arrays.asList(byteBuffers).stream()
                        .map(buffer -> "position=" + buffer.position() + ", limit=" + buffer.limit())
                        .forEach(System.out::println);
            }

            //将所有的buffer进行flip
            Arrays.asList(byteBuffers).forEach(buffer -> {
                buffer.flip();
            });

            //将数据读出显示到客户端
            long byteWrite = 0;
            while (byteWrite < messageLength) {
                long l = socketChannel.write(byteBuffers);
                byteWrite += l;
            }

            //将所有的bufer进行clear
            Arrays.asList(byteBuffers).forEach(buffer -> {
                buffer.clear();
            });

            System.out.println("byteRead=" + byteRead + " byteWrite=" + byteWrite);
        }
    }
}
```

