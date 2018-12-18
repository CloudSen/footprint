[TOC]

# NIO Channel Transfer简介

Java NIO 支持Channel之间直接传输数据。例如现在使用的是 `FileChannel` ，那么它就可以使用 `transferTo()` 方法和 `transferFrom()` 方法来完成Channel之间的通信。  



# transferFrom()

`transferFrom()` 方法能将其他Channel中的数据传到本Channel。它接收三个参数：  

1. `src` - 提供数据的源头channel；
2. `position` - 指定数据传输的起始位置，不能为负数；
3. `count` - 指定能传输的最大字节数，不能为负数。

示例代码如下：  

```java
public void transferFromTest() throws Exception {
    try (
        FileChannel fromChannel = new RandomAccessFile("/home/cloudsen/work/java/learning/java11-review/src/umbrella/nio/nio-data.txt",
                                                       "rw").getChannel();
        FileChannel toChannel = new RandomAccessFile("/home/cloudsen/work/java/learning/java11-review/src/umbrella/nio/nio-data-w.txt",
                                                     "rw").getChannel()
    ) {
        long position = 0;
        long count = fromChannel.size();
        toChannel.transferFrom(fromChannel, position, count);
    }
}
```

此外，一些 `SocketChannel` 的具体实现类可能只传输其内部缓冲区中准备好的数据。因此，从SocketChannel传输到FileChannel也许不会传输 `count` 指定大小的整个数据。  

# transferTo()

`transferTo()` 方法能将本Channel中的数据传输给其他Channel。它接收三个参数：  

1. `position` - 指定数据传输的起始位置，不能为负数；
2. `count` - 指定能传输的最大字节数，不能为负数；
3. `target` - 目标Channel。

示例代码如下：  

```java
public void transferToTest() throws Exception {
    try (
        FileChannel channel = new RandomAccessFile("/home/cloudsen/work/java/learning/java11-review/src/umbrella/nio/nio-data.txt",
                                                   "rw").getChannel();
        FileChannel toChannel = new RandomAccessFile("/home/cloudsen/work/java/learning/java11-review/src/umbrella/nio/nio-data-w.txt",
                                                     "rw").getChannel()
    ) {
        long position = 0;
        long count = channel.size();
        channel.transferTo(position, count, toChannel);
    }
}
```

# 参考资料

1. [Java NIO Tutorial - Channel To Channel Transfers](http://tutorials.jenkov.com/java-nio/channel-to-channel-transfers.html)

