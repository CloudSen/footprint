[TOC]

# Channel简介

通常，NIO中的所有IO都起始于Channel。Java NIO和Stream相似，但有以下几点不同：  

- 你将数据读取和写入Channel，但Stream通常是单向的（读或写）
- Channel可以异步读写
- 数据是从Channel写入Buffer，或从Buffer取出到Channel

![FEp8oT.png](https://s1.ax1x.com/2018/11/27/FEp8oT.png)  

# Channel实现类

以下是主要的Channel实现类：  

- FileChannel
- DatagramChannel
- SocketChannel
- ServerSocketChannel

正如你所看到的，这些Channel包含了 UDP/TCP网络IO以及文件IO。  

`FileChannel` - 将数据从文件中读取或写入文件。  

`DatagramChannel` - 通过UDP从网络中读取或写入数据。  

`SocketChannel`  - 通过TCP从网络中读取或写入数据。  

`ServerSocketChannel`  - 就像web服务器一样，监听TCP连接。每个连接，都会创建一个SocketChannel。  



# 简单应用

以下案例，使用 `FileChannel` 将文件数据写入Buffer：  

```java
package nio;

import java.io.RandomAccessFile;
import java.nio.ByteBuffer;
import java.nio.channels.FileChannel;

public class ChannelTest {

    public void fileChannelTest() throws Exception {
        RandomAccessFile aFile = new RandomAccessFile("D:\\code\\Java\\src\\nio\\nio-data.txt", "rw");
        // 创建一个文件通道
        FileChannel readChannel = aFile.getChannel();
        // 创建内存空间大小为5的字节缓存区
        ByteBuffer buf = ByteBuffer.allocate(5);
        // 通过文件通道，读取文件中的数据，写到缓存区
        int bytesRead = readChannel.read(buf);

        while (bytesRead != -1) {
            System.out.println("Read: " + bytesRead);
            // 缓存区写模式转为读模式，设置limit等于当前的位置position，然后position置为0，capacity不变
            buf.flip();

            while (buf.hasRemaining()) {
                System.out.println((char) buf.get());
            }
            // 缓冲区读模式转换为写模式，在当前的位置，将position设为0，limit设置为与capacity相同
            buf.clear();
            bytesRead = readChannel.read(buf);
        }
        aFile.close();
    }
}
```

过程很简单，首先创建了一个文件Channel，然后通过该Channel读取文件数据并写入Buffer缓存区，然后调用flip()方法后，将当前Buffer中的内容再读出来。



# 参考资料

1. [Java NIO Tutorial - Channel](http://tutorials.jenkov.com/java-nio/channels.html)