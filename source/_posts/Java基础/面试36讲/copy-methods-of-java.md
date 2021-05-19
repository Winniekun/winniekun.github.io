---
title: Java的几种文件拷贝方式
date: 2020-05-14 11:18:51
tags:
	- java基础
	- 面试36讲
categories: java
thumbnail:
---



## 前言

<!--more-->

Java中文件的拷贝方式，大体上分为两个派系

1. stream的方式实现拷贝
2. NIO的方式实现拷贝

以下举例四个经典的拷贝方式的实现，尽管Java中有自己的Copy实现，还是想使用BIO或NIO实现下面的四个方式



## 四种拷贝方式

### 无Buffer的输入输出流拷贝方式

```java
public class NoBufferStreamCopy implements FileCopyRunner {
    @Override
    public void copyFile(File source, File target) {
        try (
                InputStream input = new FileInputStream(source);
                OutputStream output = new FileOutputStream(target);
        ) {
            int read;
            while ((read = input.read())!= -1){
                output.write(read);
            }
        } catch (FileNotFoundException e) {
            e.printStackTrace();
        } catch (IOException e) {
            e.printStackTrace();
        }

    }

    @Override
    public String toString() {
        return "NoBufferStreamCopy";
    }
}
```



### 有Buffer的输入输出流拷贝方式

```java
public class BufferStreamCopy implements FileCopyRunner {
    @Override
    public void copyFile(File source, File target) {
        try (
                InputStream input = new FileInputStream(source);
                OutputStream output = new FileOutputStream(target);
        ) {
            byte[] buffer = new byte[1024];
            int leng;
            while ((leng = input.read(buffer)) != -1){
                output.write(buffer, 0, leng);
            }

        } catch (FileNotFoundException e) {
            e.printStackTrace();
        } catch (IOException e) {
            e.printStackTrace();
        }
    }

    @Override
    public String toString() {
        return "BufferStreamCopy";
    }
}
```



### NIO基础的拷贝实现

```java
public class NioBufferCopy implements FileCopyRunner {
    @Override
    public void copyFile(File source, File target) {
        try (
                FileChannel input = new FileInputStream(source).getChannel();
                FileChannel output = new FileOutputStream(target).getChannel();
        ) {
            ByteBuffer buffer = ByteBuffer.allocate(1024);
            while (input.read(buffer) != -1){
                buffer.flip();
                while (buffer.hasRemaining()) {
                    output.write(buffer);
                }
                buffer.clear();
            }
        } catch (FileNotFoundException e) {
            e.printStackTrace();
        } catch (IOException e) {
            e.printStackTrace();
        }
    }

    @Override
    public String toString() {
        return "NioBufferCopy";
    }
}
```



### NIO Transfer实现拷贝 

```java
public class NioTransferCopy implements FileCopyRunner {
    @Override
    public void copyFile(File source, File target) {
        try (
                FileChannel inputChannel = new FileInputStream(source).getChannel();
                FileChannel ouputChannel = new FileOutputStream(target).getChannel();
        ) {
            for (long count = inputChannel.size(); count > 0;) {
                long transferTo = inputChannel.transferTo(inputChannel.position(), count, ouputChannel);
                count -= transferTo;

            }
        } catch (FileNotFoundException e) {
            e.printStackTrace();
        } catch (IOException e) {
            e.printStackTrace();
        }
    }

    @Override
    public String toString() {
        return "NioTransferCopy";
    }
}
```

### 对比

#### 测试类

```java
public class FileCopyDemo {
    private static final int ROUNDS = 5;
    private static final File sourceBig = new File("/home/kongweikun/Downloads/test/ccc.txt");
    private static final File sourceMid = new File("/home/kongweikun/Downloads/test/bbb.txt");
    private static final File sourceSmal = new File("/home/kongweikun/Downloads/test/aaa.txt");
    private static final File target = new File("/home/kongweikun/Downloads/test/ddd.txt");

    public static void benchMark(FileCopyRunner test, File source, File target, String name) {
        long elapsed = 0L;
        for (int i = 0; i < ROUNDS; i++) {
            long start = System.currentTimeMillis();
            test.copyFile(source, target);
            elapsed = System.currentTimeMillis() - start;
            // 删除copy之后的文件
            target.delete();
        }
        System.out.println(test + name + " : " + elapsed / ROUNDS + "毫秒");
    }

    public static void main(String[] args) {
        FileCopyRunner noBufferStreamCopy = new NoBufferStreamCopy();
        FileCopyRunner bufferedStreamCopy = new BufferStreamCopy();
        FileCopyRunner nioBufferCopy = new NioBufferCopy();
        FileCopyRunner nioTransferCopy = new NioTransferCopy();

        // 测试 1
        System.out.println("-----------------------------------------");
        System.out.println("无Buffer的BIO");
        benchMark(noBufferStreamCopy, sourceBig, target, "大文件");
        benchMark(noBufferStreamCopy, sourceMid, target, "中等文件");
        benchMark(noBufferStreamCopy, sourceSmal, target, "小文件");
        System.out.println("-----------------------------------------");
        System.out.println("有Buffer的BIO");
        benchMark(bufferedStreamCopy, sourceBig, target, "大文件");
        benchMark(bufferedStreamCopy, sourceMid, target, "中等文件");
        benchMark(bufferedStreamCopy, sourceSmal, target, "小文件");
        //
        System.out.println("-----------------------------------------");
        System.out.println("NIO基础");
        benchMark(nioBufferCopy, sourceBig, target, "大文件");
        benchMark(nioBufferCopy, sourceMid, target, "中等文件");
        benchMark(noBufferStreamCopy, sourceSmal, target, "小文件");
        System.out.println("-----------------------------------------");
        System.out.println("NIO的TransferTo");
        benchMark(nioTransferCopy, sourceBig, target, "大文件");
        benchMark(nioTransferCopy, sourceMid, target, "中等文件");
        benchMark(nioTransferCopy, sourceSmal, target, "小文件");
    }
}
```



```tex
-----------------------------------------
无Buffer的BIO
NoBufferStreamCopy大文件 : 614毫秒
NoBufferStreamCopy中等文件 : 0毫秒
NoBufferStreamCopy小文件 : 0毫秒
-----------------------------------------
有Buffer的BIO
BufferStreamCopy大文件 : 1毫秒
BufferStreamCopy中等文件 : 0毫秒
BufferStreamCopy小文件 : 0毫秒
-----------------------------------------
NIO基础
NioBufferCopy大文件 : 1毫秒
NioBufferCopy中等文件 : 0毫秒
NoBufferStreamCopy小文件 : 0毫秒
-----------------------------------------
NIO的TransferTo
NioTransferCopy大文件 : 0毫秒
NioTransferCopy中等文件 : 0毫秒
NioTransferCopy小文件 : 0毫秒
```



## 总结

对于 Copy 的效率,这个其实与操作系统和配置等情况相关,总体上来说,NIOtransferTo/From 的方式可能更快,因为它更能利用现代操作系统底层机制,避免不必要拷贝和上下文切换。



### 不同的 copy 方式,底层机制有什么区别?

这里涉及到了用户态空间和内核态空间,这是操作系统层面的基本概念,操作系统内核、硬件驱动等运行在内核态空间,具有相对高的特权.而用户态空间,则是给普通应用和服务使用.

当我们使用输入输出流的进行读写的时候,实际上经历了多次的上下文切换:

**应用数据的读取**: 首先内核态将数据从磁盘读取到内核缓存,再切换到用户态将数据从内核缓存读取到用户缓存

![用户态-内核态](https://i.loli.net/2020/05/22/FZw5oc3VjxGby2e.png)

所以,这种方式会带来一定的额外开销,可能会降低 IO 效率。

但是NIO的transferTo的实现, 会涉及到零拷贝, 也就是数据传输并不会涉及到用户态,省去了上下文的切换和不必要的拷贝.

transferTo的传输过程是:

![transferTo传输过程]()