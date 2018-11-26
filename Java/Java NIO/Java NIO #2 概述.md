[TOC]

Java NIO 由 `Channels` 、`Buffers` 和 `Selectors` 这三个组件构成 ，其中Selector是真正的核心。像 `Pipe` 和 `FileLock` 只是用来结合这三个核心组件的实用类。  

# Channels和Buffers

通常，NIO中的所有IO都起始于Channel。Channel有点类似于stream，Channel中的数据能够被Buffer读取，数据也能从Buffer中写入Channel。  

![FEp8oT.png](https://s1.ax1x.com/2018/11/27/FEp8oT.png)  

以下是主要的Channel实现类：  

- FileChannel
- DatagramChannel
- SocketChannel
- ServerSocketChannel

正如你所看到的，这些Channel包含了 UDP/TCP网络IO以及文件IO。  

以下是主要的Buffer实现类：  

- ByteBuffer
- CharBuffer
- DoubleBuffer
- FloatBuffer
- IntBuffer
- LongBuffer
- ShortBuffer
- MappedByteBuffer

正如你所看到的，这些Buffer包含了可以通过IO发送的基本数据类型：byte, short, int, long, float, double, characters。`MappedByteBuffer`  是用来与内存映射文件一起使用的。  

# Selectors

Selector允许单线程处理多个Channel。它非常适用于程序打开了多个Channel连接，但每个连接上的流量占用都很低这样的情况，例如，聊天服务器。  

![FEp3wV.png](https://s1.ax1x.com/2018/11/27/FEp3wV.png)  

要使用Selector，首先使用它注册Channel。然后调用它的select()方法。此方法将阻塞，直到Channel事件准备就绪。一旦该方法返回，该线程就可以处理这些事件。例如传入连接，接收数据等事件。





