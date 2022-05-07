---
title: "My First Post"
date: 2022-05-04T12:33:23+08:00
draft: false
tags: ["标签一","标签二"]
---

## 引言

如果不了解阻塞和非阻塞，同步和异步，请先阅读此文章 [五种网络IO模型详解](https://www.notion.so/IO-abb51751b8f1402aa1161b1a81730a65)

NIO是一种同步非阻塞的网络IO模型，传统的BIO一般通过单独创建线程或者线程池的方式创建线程处理请求，这样会带来以下问题：

- 线程池大小有限，每个请求都要创建一个线程，无法处理大量的请求
- 线程上下文切换十分耗费CPU资源

NIO对此做了优化，使用一个Selector（多路复用器）主线程来负责IO事件分发，这样一个主线程就可以接入成千上万的客户端连接，而且性能不会随着客户端连接的增加而下降

![netty架构图](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/ec2d9640-db80-4f1b-826c-186694f2ac34/Untitled.png)

netty架构图

Netty使用Selector同时监视多个通道，可以注册多个通道到同一个选择器上，然后使用一个单独的线程来“选择”已经就绪的通道。这种“选择”机制为一个单独线程管理多个通道提供了可能。

接下来简单说说BIO和NIO的区别：

- NIO中Channel是全双工(是说可以通过Channel 即可完成读操作，也可以完成写操作)，而BIO则需要读写操作需要分别创建一个线程
- BIO是面向流的，每次需要读取所有的字节，不能移动流中的数据；NIO是面向缓冲的，数据每次数据会被读取到一个缓冲区，需要时可以在缓冲区中前后移动处理，这增加了处理过程的灵活性

## JDK NIO架构分析

Java 中的 NIO 指的是从 1.4 版本后，提供的一套可以替代标准的 Java IO 的 new API。有三部分组成

- Buffer 缓冲区
- Channel 通道
- Selector 选择器

```java
// 开启一个channel
ServerSocketChannel serverSocketChannel = ServerSocketChannel.open();
// 设置为非阻塞
serverSocketChannel.configureBlocking(false);
// 绑定端口
serverSocketChannel.bind(new InetSocketAddress(PORT));
// 打开一个多路复用器
Selector selector = Selector.open();
// 绑定多路复用器和channel
serverSocketChannel.register(selector, SelectionKey.OP_ACCEPT);
// 获取到达的事件
while (selector.select() > 0) {
    Set<SelectionKey> keys = selector.keys();
    Iterator<SelectionKey> iterator = keys.iterator();
    while (iterator.hasNext()) {
        SelectionKey selectionKey = iterator.next();
        if (selectionKey.isAcceptable()) {
            // 处理逻辑
        }
        if (selectionKey.isReadable()) {
            // 处理逻辑
        }
    }
}
```

<aside> 💡 大致的流程是创建一个channel并绑定到selector，并通过轮询selector中的channel进行相关操作

</aside>

### 单线程案例（Selector模式）

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/f19b7aed-625d-48c2-9586-b61229e85d7c/Untitled.png)

[NIO-sigleThread](https://www.notion.so/NIO-sigleThread-ae1cfb3ae5bd4c86a0229eb2b3d097e7)

### 多线程案例

单线程中所有客户端的IO读写都为一个线程，可以借鉴BIO，使用线程池或者单独创建线程的方式处理读写任务

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/feaacdda-9b35-45ef-b70b-a50f361ba7b8/Untitled.png)

## Reactor模式案例

> Reactor不是nio中的某个类，而是一种设计模式，是一种事件处理模式，能够以此处理一个或者多个输入，并将请求进行多路分解和同步分派

