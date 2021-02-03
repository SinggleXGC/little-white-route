# Channel

## 什么是Channel

NIO的通道类似流，但是有些不同：

1. NIO可以同时进行读和写，流只能读或写
2. 通道可以实现异步读写数据
3. 通道能够从缓冲读取数据，也可以写数据到缓冲



Channel在NIO中是一个接口。常见的Channel类有：

FileChannel、DatagramChannel、ServerSocketChannel、SocketChannel

FileChannel用于文件数据的读写，DatagramChannel用于UDP数据的读写，ServerSocketChannel和SocketChannel用于TCP数据的读写。



## FileChannel类

FileChannel主要对本地文件进行IO操作，常见方法有：

```java
public int read(ByteBuffer dst) //从通道读取数据并放到缓冲区
public int write(ByteBuffer src) //从缓冲区读取数据写入到通道
//从目标通道中复制数据到当前通道
public long transferFrom(ReadableByteChannel src, long position, long count)
//把数据从当前通道复制给目标通道
public long transferTo(long position, long count, WritableByteChannel target)
```



## 



## 示例

### 将数据写入文件

```java
/**
 * 将数据写入到文件
 */
public class NIOFileChannel01 {
    public static void main(String[] args) throws IOException {

        String str = "hello world";
        //创建一个输出流
        FileOutputStream fileOutputStream =
                new FileOutputStream(NIOFileChannel01.class.getClassLoader().getResource("a.txt").getFile());
        //从输出流获取FileChannel对象
        FileChannel fileChannel = fileOutputStream.getChannel();

        //创建ByteBuffer
        ByteBuffer byteBuffer = ByteBuffer.allocate(1024);
        byteBuffer.put(str.getBytes());
        byteBuffer.flip();

        //把ByteBuffer的数据写到fileChannel
        fileChannel.write(byteBuffer);
        fileOutputStream.close();
    }
}
```



### 从文件读取数据

```
/**
 * 从文件读取数据
 */
public class NIOFileChannel02 {

    public static void main(String[] args) throws IOException {
        //创建文件输输入流
        FileInputStream fileInputStream =
                new FileInputStream(NIOFileChannel02.class.getClassLoader().getResource("a.txt").getFile());
        FileChannel fileChannel = fileInputStream.getChannel();

        ByteBuffer byteBuffer = ByteBuffer.allocate(1024);
        //从fileChannel中读取数据到ByteBuffer
        fileChannel.read(byteBuffer);
        fileInputStream.close();

        System.out.println(new String(byteBuffer.array()));
    }
}
```



### 完成文件拷贝(普通)

```java
/**
 * 完成文件拷贝
 * 将1.txt 复制为 2.txt
 */
public class NIOFileChannel03 {

    public static void main(String[] args) throws IOException {
        FileInputStream fileInputStream = new FileInputStream("1.txt");
        FileChannel fileChannel01 = fileInputStream.getChannel();

        FileOutputStream fileOutputStream = new FileOutputStream("2.txt");
        FileChannel fileChannel02 = fileOutputStream.getChannel();

        ByteBuffer byteBuffer = ByteBuffer.allocate(1024);

        while (true) {
            byteBuffer.clear();
            int read = fileChannel01.read(byteBuffer);
            if (read == -1) {
                break;
            }
            byteBuffer.flip();
            fileChannel02.write(byteBuffer);
        }
        fileInputStream.close();
        fileOutputStream.close();
    }
}
```



### 完成文件拷贝(transferFrom方法)

```java
/**
 * 用FileChannel和transferFrom方法完成图片的拷贝
 */
public class NIOFileChannel04 {

    public static void main(String[] args) throws IOException {
        FileInputStream fileInputStream = new FileInputStream("a.jpeg");
        FileChannel fileChannel01 = fileInputStream.getChannel();

        FileOutputStream fileOutputStream = new FileOutputStream("a2.jpeg");
        FileChannel fileChannel02 = fileOutputStream.getChannel();

        fileChannel02.transferFrom(fileChannel01, 0, fileChannel01.size());

        fileChannel01.close();
        fileChannel02.close();
        fileInputStream.close();
        fileOutputStream.close();

    }
}
```



## ServerSocketChannel

ServertSocketChannel在服务器端监听新的客户端连接

相关方法

```java
public abstract class ServerSocketChannel extends AbstractSelectableChannel
    implements NetworkChannel{
    
    public static ServerSocketChannel open();
    public final ServerSocketChannel bind(SocketAddress local);
    public final SelectableChannel configureBlocking(boolean block)
    public final SocketChannel accept()
	public final SelectionKey register(Selector sel, int ops)
}
```



## SocketChannel

SocketChannel，网络IO通道，具体负责进行读写操作。NIO把缓冲区的数据写入通道，或者把通道里的数据读到缓冲区。

```java
public abstract class SocketChannel extends AbstractSelectableChannel
    implements ByteChannel, ScatteringByteBuffewr, GatheringByteChannel, NetworkChannel {
    
    public static SocketChannel open();
    public final SelectableChannel configureBlocking(boolean false);
    public boolean connect(SocketAddress remote);
    public boolean finishConnect();
    public int write(ByteBuffer src);
    public int read(ByteBuffer dest);
    public final SelectionKey register(Selector sel, int ops, Object att);
    public final void close();
}
```

