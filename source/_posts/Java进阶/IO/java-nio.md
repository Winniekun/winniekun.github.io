---
title: JavaIO的今世-NIO
date: 2020-04-07 14:16:56
tags: I/O
categories: java 
thumbnail:
---

## Java NIO

新的输入/输出 (NIO) 库是在 JDK 1.4 中引入的，弥补了原来的 I/O 的不足，提供了高速的、面向块的 I/O。

<!--more-->

## BIO带来的挑战

BIO(blocking-IO)

> 不管是磁盘 I/O 还是网络 I/O，数据在写入 OutputStream 或者从 InputStream 读取时都有可能会阻塞。一旦有线程阻塞将会失去 CPU 的使用权，这在当前的大规模访问量和有性能要求情况下是不能接受的。虽然当前的网络 I/O 有一些解决办法，如一个客户端一个处理线程，出现阻塞时只是一个线程阻塞而不会影响其它线程工作，还有为了减少系统线程的开销，采用线程池的办法来减少线程创建和回收的成本，但是有一些使用场景仍然是无法解决的。如当前一些需要大量 HTTP 长连接的情况，像淘宝现在使用的 Web 旺旺项目，服务端需要同时保持几百万的 HTTP 连接，但是并不是每时每刻这些连接都在传输数据，这种情况下不可能同时创建这么多线程来保持连接。即使线程的数量不是问题，仍然有一些问题还是无法避免的。如这种情况，我们想给某些客户端更高的服务优先级，很难通过设计线程的优先级来完成，另外一种情况是，我们需要让每个客户端的请求在服务端可能需要访问一些竞争资源，由于这些客户端是在不同线程中，因此需要同步，而往往要实现这些同步操作要远远比用单线程复杂很多。以上这些情况都说明，我们需要另外一种新的 I/O 操作方式。



## NIO 简介

NIO 是一种同步非阻塞的 I/O 模型，在 Java 1.4 中引入了 NIO 框架，对应 `java.nio` 包，提供了 `Channel` 、`Selector`、`Buffer` 等抽象。

NIO有两种解释：一种叫非阻塞IO（Non-blocking I/O），另一种叫新的IO（New I/O），其实是同一个概念。它是一种同步非阻塞的I/O模型，也是I/O多路复用的基础，已经被越来越多地应用到大型应用服务器，成为解决高并发与大量连接、I/O处理问题的有效方式。(**只有Socket Channel 才能配置为非阻塞，而 FileChannel 不能， 为 FileChannel 配置非阻塞也没有意义**)

NIO是一种基于`通道`和`缓冲区`的I/O方式，它可以使用Native函数库直接分配堆外内存（区别于JVM的运行时数据区），然后通过一个存储在Java堆里面的`DirectByteBuffer`对象作为这块内存的直接引用进行操作。这样能在一些场景显著提高性能，因为避免了在Java堆和Native堆中来回复制数据。



## Channel

`Channel`是对 BIO 中的流的模拟，可以通过它读写数据(**真正读写**数据的是Buffer)，永远不会出现直接向Channel写入数据或者直接从Channel中读取数据的操作。

通道与流的不同之处在于：

- **流是单向的** - 一个流只能单纯的负责读或写。
- **通道是双向的** - 一个通道可以同时用于读写。

因为**Channel**是双向的，其能更好的反应底层操作系统的真实情况（如操作系统中的通道）。同时**Channel**包括以下类型(标粗的是常用的类型)：

- **FileChannel**：从文件中读写数据；
- `DatagramChannel`：通过 UDP 读写网络中数据；
- **SocketChannel**：通过 TCP 读写网络中数据；
- **ServerSocketChannel**：可以监听新进来的 TCP 连接，对每一个新进来的连接都会创建一个 SocketChannel。

### 基本使用

1. 从**Channel**进行读取数据
   
   1. 首先会创建一个**Buffer**，**Channel**将数据读入其中，之后在**Bufer**中读取数据
   
      ```java
      public static void read() throws IOException {
          FileInputStream fileInputStream = new FileInputStream("aaa.txt");
          FileChannel inputChannel = fileInputStream.getChannel();
          ByteBuffer buffer = ByteBuffer.allocate(1024);
          inputChannel.read(buffer);
          // 切换为读模式
          buffer.flip();
          while (buffer.hasRemaining()){
              byte b = buffer.get();
              System.out.println("Character: " + (char)b);
          }
          buffer.clear();
          fileInputStream.close();
      }
      ```
   
2. 向**Channel**写入数据
   
   1. 创建**Buffer**，填充数据。之后**Channel**去写出数据
   
      ```java
      public static void write() throws IOException {
          FileOutputStream fileOutputStream = new FileOutputStream("bbb.txt");
          FileChannel outChannel = fileOutputStream.getChannel();
          ByteBuffer buffer = ByteBuffer.allocate(1024);
          byte[] msg = "hello i'm wkk".getBytes();
          buffer.put(msg);
          buffer.flip();
          outChannel.write(buffer);
          fileOutputStream.close();
      }
      ```
   
      



## Buffer

> ```
> A buffer is a linear, finite sequence of elements of a specific
> * primitive type.  Aside from its content, the essential properties of a
> * buffer are its capacity, limit, and position:    --- 摘自Buffer源码
> ```

**Buffer**本身就是一块内存，底层实现上，实际是一个数组。数据的读写均是通过**Buffer**实现。

```java
public class NioBase {
    public static void main(String[] args) {
        // 初始化一个Buffer
        IntBuffer buffer = IntBuffer.allocate(1024);

        // 向Buffer中随机写入10个数字
        for (int i = 0; i < 10; i++) {
            int randomNumber = new SecureRandom().nextInt(20);
            buffer.put(randomNumber);
        }

        // 切换为读模式
        buffer.flip();
        while (buffer.hasRemaining()){
            System.out.print(buffer.get() + " ");
        }

    }
}
```



### Capacity、Position、Limit

> ```
> *   <p> A buffer's <i>capacity</i> is the number of elements it contains.  The
> *   capacity of a buffer is never negative and never changes.  </p>
> *
> *   <p> A buffer's <i>limit</i> is the index of the first element that should
> *   not be read or written.  A buffer's limit is never negative and is never
> *   greater than its capacity.  </p>
> *
> *   <p> A buffer's <i>position</i> is the index of the next element to be
> *   read or written.  A buffer's position is never negative and is never
> *   greater than its limit.  </p>
> ```



在NIO中，真正和数据打交道的是`Buffer`。可以将`Buffer`简单的理解为一组基本数据类型的元素列表，其通过以下的基本变量来保存这个数据的当前位置状态，共有四个索引（mark用于记录以下三个索引，所以没有记录）。

| 索引     | 说明                                                   |
| -------- | ------------------------------------------------------ |
| capacity | 缓冲数组的总长度                                       |
| position | 下一个要操作的数据元素的位置                           |
| limit    | 缓冲区数组不可操作的下一个元素的位置， limit<=capacity |

![image](https://i.loli.net/2020/04/24/UyzdCTEPgiutxR1.png)

#### demo

```java
public static void main(String[] args) {
    // 创建缓冲区
    ByteBuffer buffer = ByteBuffer.allocate(1024);
    //buffer中四个变量的值
    System.out.println("初始时---->limit---->" + buffer.limit());
    System.out.println("初始时---->capacity---->" + buffer.capacity());
    System.out.println("初始时---->position---->" + buffer.position());
    System.out.println("初始时---->mark---->" + buffer.mark());
    System.out.println("--------------------------------");
//初始时---->limit---->1024
//初始时---->capacity---->1024
//初始时---->position---->0
//初始时---->mark---->java.nio.HeapByteBuffer[pos=0 lim=1024 cap=1024]
    String s = "abcd";
    buffer.put(s.getBytes());
    //buffer添加数据之后，四个变量的值
    System.out.println("put完之后---->limit---->" + buffer.limit());
    System.out.println("put完之后---->capacity---->" + buffer.capacity());
    System.out.println("put完之后---->position---->" + buffer.position());
    System.out.println("put完之后---->mark---->" + buffer.mark());
//put完之后---->limit---->1024
//put完之后---->capacity---->1024
//put完之后---->position---->4
//put完之后---->mark---->java.nio.HeapByteBuffer[pos=4 lim=1024 cap=1024]
    buffer.flip();
    System.out.println("-------------------------------------------");
    System.out.println("flip()之后---->limit---->" + buffer.limit());
    System.out.println("flip()之后---->capacity---->" + buffer.capacity());
    System.out.println("flip()之后---->position---->" + buffer.position());
    System.out.println("flip()之后---->mark---->" + buffer.mark());
    
    byte[] bytes = new byte[buffer.limit()];
    buffer.get(bytes);
    System.out.println(new String(bytes, 0, bytes.length));
//flip()之后---->limit---->4
//flip()之后---->capacity---->1024
//flip()之后---->position---->0
//flip()之后---->mark---->java.nio.HeapByteBuffer[pos=0 lim=4 cap=1024]
//abc
    System.out.println("-------------------------------------------");
    System.out.println("读完之后---->limit---->" + buffer.limit());
    System.out.println("读完之后---->capacity---->" + buffer.capacity());
    System.out.println("读完之后---->position---->" + buffer.position());
    System.out.println("读完之后---->mark---->" + buffer.mark());
//读完之后---->limit---->4
//读完之后---->capacity---->1024
//读完之后---->position---->3
//读完之后---->mark---->java.nio.HeapByteBuffer[pos=3 lim=4 cap=1024]    
    System.out.println("----------------清空缓冲区-------------");
    buffer.clear();
    System.out.println("清空之后---->limit---->" + buffer.limit());
    System.out.println("清空之后---->capacity---->" + buffer.capacity());
    System.out.println("清空之后---->position---->" + buffer.position());
    System.out.println("清空之后---->mark---->" + buffer.mark());
//清空之后---->limit---->1024
//清空之后---->capacity---->1024
//清空之后---->position---->0
//清空之后---->mark---->java.nio.HeapByteBuffer[pos=0 lim=1024 cap=1024]
}
```

关于这三种索引的实际关系，以及`flip()` `clear()`在下述`Buffer的读写`中会详细阐述

### Buffer 类型

Buffer实现了java的所有的基础数据类型

* ByteBuffer
* ShortBuffer
* IntBuffer
* LongBuffer
* FloatBuffer
* DoubleBuffer
* CharBuffer
* **MappedByteBuffer**

使用`Buffer`对象之前需要先进行分配，每个类型的`Buffer`类都可以使用静态方法`allocate（）`分配该`Buffer`的容量大小

### 读和写

#### 向Buffer中写入数据：

有两种方式：

1. 通过channel.read(buffer)写入数据

   ```java
   RandomAccessFile file = new RandomAccessFile("../../Downloads/aaa.txt", "rw");
   FileChannel inChannel = file.getChannel();
   // 创建缓冲区
   ByteBuffer buffer = ByteBuffer.allocate(1024);
   // 从channel中写入数据
   int read = inChannel.read(buffer);
   ```

   

2. 直接通过buffer.put()写入数据

   ```java
   // 创建缓冲区
   ByteBuffer buffer = ByteBuffer.allocate(18);
   String s = "abcd";
   buffer.put(s.getBytes());
   ```

**`flip()`方法**

使用该方法之后，`position`索引会指向0，`limit`索引会指向缓冲区数组不可操作的下一个元素的位置（也可表示为当前缓冲数组可读元素的长度）

![flip__方法索引变化.png](https://i.loli.net/2020/04/24/9qBn1RE8dsIYULt.png)

所以，flip()方法也可以理解为切换读模式，输入一定的元素之后，感觉需要读出数据，使用flip()方法，切换为读的模式。

#### 从Buffer中读出数据

同理，也有两种方式从**Buffer**中读出数据

1. channel.write(buffer)
2. buffer.get()

`clear()`方法，操作数据的索引重归初始状态，类似于数据被清空，可以理解为切换写模式

## Selector

用于监听多个通道的事件（比如：连接打开，数据到达）。因此，单个的线程可以监听多个数据通道。即用选择器，借助单一线程，就可对数量庞大的活动I/O通道实施监控和维护。其和单片机中的事件轮询是一个道理,可惜本科上单片机的时候,并没有好好听(对不起小邹邹)

> NIO 常常被叫做非阻塞 IO，主要是因为 NIO 在网络通信中的非阻塞特性被广泛使用。
>
> NIO 实现了 IO 多路复用中的 Reactor 模型，一个线程 Thread 使用一个选择器 Selector 通过**轮询的方式** 去监听多个通道 Channel 上的事件，从而让一个线程就可以处理多个事件。
>
> 通过配置监听的通道 Channel 为**非阻塞**，那么当 Channel 上的 IO 事件还未到达时， 就不会进入阻塞状态一直等待，而是继续轮询其它 Channel，找到 IO 事件已经到达的 Channel 执行。
>
> 因为创建和切换线程的开销很大，因此使用一个线程来处理多个事件而不是一个线程处理一个事件， 对于 IO 密集型的应用具有很好地性能。
>
> 应该注意的是，只有Socket的Channel 才能配置为非阻塞，而 FileChannel 不能， 为 FileChannel 配置非阻塞也没有意义。

**使用Selector的优点:** 使用更少的线程来处理任务, 相比更多的线程,避免了线程上下文的切换开销

![](https://i.loli.net/2020/05/21/682eHdpv3hiy7sE.png)

可以将上图做更细致的绘制:

![](https://i.loli.net/2020/05/22/O384LvYwqpg91Uf.png)

>  NIO模型的服务器端如何实现非阻塞？服务器上所有`Channel`需要向`Selector`注册，而`Selector`则负责监视这些`Channel`的IO状态(`观察者`)，当其中任意一个或者多个`Channel`具有可用的IO操作时，该`Selector`的`select()`方法将会返回大于0的整数(表示该`Selector`上有多少个`Channel`具有可用的IO操作)，并提供了`selectedKeys()`方法来返回这些`Channel`对应的`SelectionKey`集合(一个`SelectionKey`对应一个就绪的通道)。正是通过`Selector`，使得服务器端只需要不断地调用`Selector`实例的`select()`，即可知道当前所有`Channel`是否有需要处理的IO操作。

### 步骤

#### 创建选择器

```java
Selector selector = Selector.open();
```



#### 绑定端口,通道注册到选择器

```java
// 绑定端口
ServerSocketChannel server = ServerSocketChannel.open();
server.configureBlocking(false); // 设置为非阻塞
ServerSocket socket = server.socket();
socket.bind(new InetSocketAddress(port));

// 将通道注册到该线程的选择器
server.register(selector, SelectionKey.OP_ACCEPT);

```

通道必须配置为非阻塞模式，否则使用选择器就没有任何意义了，因为如果通道在某个事件上被阻塞，那么服务器就不能响应其它事件，必须等待这个事件处理完毕才能去处理其它事件，显然这和选择器的作用背道而驰。

在将通道注册到选择器上时，还需要指定要注册的具体事件，主要有以下几类：

- SelectionKey.OP_CONNECT
- SelectionKey.OP_ACCEPT
- SelectionKey.OP_READ
- SelectionKey.OP_WRITE



#### 监听事件

```java
selector.select();
```

使用 select() 来监听到达的事件，它会**一直阻塞直到有至少一个事件到达**。

#### 获取可达事件

```java
// 可达事件
Set<SelectionKey> selectionKeys = selector.selectedKeys();
Iterator<SelectionKey> iterator = selectionKeys.interator();
while(iterator.hasnext()){
    SelectionKey key = iterator.next();
    if(key.isAcceptable()){
        // ...
    }else if(key.isReadable()){
        //...
    }
    // remove已经处理的事件
    iterator.remove();
}
```



#### 事件循环

因为一次 select() 调用不能处理完所有的事件，并且服务器端有可能需要一直监听事件，因此服务器端处理事件的代码一般会放在一个死循环内。

```java
while(true){
    // 监听事件
    selector.select();
    // 获取可达事件
    Set<SelectionKey> selectionKeys = selector.selectedKeys();
	Iterator<SelectionKey> iterator = selectionKeys.interator();
	while(iterator.hasnext()){
    	SelectionKey key = iterator.next();
    	if(key.isAcceptable()){
        	// ...
    	}else if(key.isReadable()){
        	//...
    	}
    	// remove已经处理的事件
    	iterator.remove();
	}
}
```



#### 整合

```java
public void start(){
    // 1. 创建选择器
    Selector selector = Selector.open();
    // 2. 绑定端口 通道注册到选择器
    ServerSocketChannel serverChannel = ServerSocketChannel.open();
    serverChannel.configureBlocking(false);
    ServerSocket socket = serverChannel.socket();
    socket.bind(new InetSocketAddress(port));
    // 通道注册到selector上
    serverChannel.register(selector, SelectionKey.OP_ACCEPT);
    
    // 事件轮询
    while(true){
        // 3 监听事件
        selector.select();
        
        // 4 获取可达事件
        Set<SelectionKey> selectionKeys = selector.selectedKeys();
        Iterator<SelectionKey> iterator = selectionKeys.iterator();
        while(iterator.hasNext()){
            SelectionKey key = itrator.next();
            if (key.isAcceptable()) {
                ServerSocketChannel server = (ServerSocketChannel) key.channel();
                // 服务器会为每个新连接创建一个 SocketChannel
                SocketChannel client = server.accept();
                client.configureBlocking(false);
                // 这个新连接的channel主要用于从客户端读取数据
                client.register(selector, SelectionKey.OP_READ);
            }else if(key.isReadable()){
                SocketChannel client = (SocketChannel)key.channel();
                // 数据的处理
                ByteBuffer buffer = ByteBuffer.allocate(1024);
                buffer.clear();
                while (channel.read(buffer) > 0);
                buffer.flip();
                channel.write(buffer);
            }
            iterator.remove();
        }
    }
}
```



## 应用

创建一个简单的服务端

```java
public static void main(String[] args) throws IOException {
	// 服务端监听5个端口
    int[] ports = new int[5];
    int base = 9000;
    for (int i = 0; i < 5; i++) {
        ports[i] = base++;
    }
    // 创建选择器
    Selector selector = Selector.open();
    // 绑定端口 通道注册到选择器
    for (int port : ports) {
        ServerSocketChannel server = ServerSocketChannel.open();
        server.configureBlocking(false);
        ServerSocket socket = server.socket();
        socket.bind(new InetSocketAddress(port));
        server.register(selector, SelectionKey.OP_ACCEPT);
        System.out.println("监听端口: " + port);
    }
    // 事件轮询
    while (true) {
        // 监听事件
        int num = selector.select();
        System.out.println("numbers: " + num);
        // 获取可达事件
        Set<SelectionKey> selectionKeys = selector.selectedKeys();
        System.out.println("可达事件集有: " + selectionKeys.size());
        for (SelectionKey selectionKey : selectionKeys) {
            System.out.println("可达事件集类型有: " + selectionKey);
        }
        // 事件处理
        Iterator<SelectionKey> iterator = selectionKeys.iterator();
        while (iterator.hasNext()) {
            SelectionKey key = iterator.next();
            if (key.isAcceptable()) {
                ServerSocketChannel server = (ServerSocketChannel) key.channel();
                SocketChannel client = server.accept();
                client.configureBlocking(false);
                client.register(selector, SelectionKey.OP_READ);
                System.out.println("获得客户端的链接" + client);
            } else if (key.isReadable()) {
                SocketChannel channel = (SocketChannel) key.channel();
                ByteBuffer buffer = ByteBuffer.allocate(1024);
                buffer.clear();
                while (channel.read(buffer) > 0);
                buffer.flip();
                channel.write(buffer);
                System.out.println("读取: " + buffer + ", 来自于" + channel);
            }
            // 记得remove
            iterator.remove();
        }
    }
}
```



## References

* [Java NIO](http://tutorials.jenkov.com/java-nio/index.html)
* [IBM NIO 入门](https://www.ibm.com/developerworks/cn/education/java/j-nio/j-nio.html)







