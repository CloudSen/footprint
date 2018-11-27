[TOC]

# JAVA NIO简介

> 扩展阅读：
>
> - RPC框架
> - C10K
>
> - UNIX 网络编程
> - epoll、select
> - IO的几种工作模式以及阻塞模型
> - Netty

Java NIO (New IO & Non-Blocking IO) 是对传统Java IO API的一个扩展。NIO提供了与标准IO不同的工作方式——非阻塞IO。它允许你的线程在等待IO流的时候，去做其他的事情，不用占用资源干耗着。  

Java NIO 由 `Channels` 、`Buffers` 和 `Selectors` 这三个组件构成 ，其中Selector是真正的核心。像 `Pipe` 和 `FileLock` 只是用来结合这三个核心组件的实用类。  



# Channels and Buffers

标准IO流API使用字节流(byte streams)和字符流(character streams)来工作。而NIO使用的是通道(channels)与缓冲区(buffer)。数据总是从通道读入缓冲区，或从缓冲区写入通道。   



# Non-blocking IO

NIO使用了非阻塞式IO模型。比如，一个线程能够通过channel将数据读入buffer。当channel还在读取数据期间，线程能够去做其他的事情。一旦数据全部读入buffer，线程就能立即返回继续读取好的数据。同理，将数据写入channel也是如上原理。  



# Selectors

**Selectors是NIO的核心**，Selector是一个能够监控多个channel事件（如连接打开，收到数据等）的对象。因此，一个单线程能够监控多个channel的数据。  



# NIO与标准IO的区别

从上面的概念中，可以看出：

- 标准IO是面向流的，而NIO是面向缓冲区的
- 标准IO是阻塞的，NIO是非阻塞的
- NIO有选择器，一个线程能监控很多通道，而标准IO没有



# 参考资料

1. [知乎：如何学习Java的NIO？](https://www.zhihu.com/question/29005375)
2. [Java NIO Tutorial](http://tutorials.jenkov.com/java-nio/index.html)