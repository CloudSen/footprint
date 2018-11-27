[TOC]

# Scatter和Gather简介

Java NIO 内置了对 Scattter(分散)和Gather(聚集)的支持。它们是用于从Channel中读取或写入数据时用到的概念。  

Scatttering Read(分散读取)：从一个Channel中读取数据，写入多个Buffer的操作。  

Gather Write(聚集写入)：从多个Buffer中读取数据，写入一个Channel中的操作。  

Scatter和Gather常用于需要将传输数据的各个部分分开处理的场景。例如，一个消息由header和body组成，你也许会将header和body分开，放入不同的Buffer中，这样能更加灵活地处理消息。  



# Scattering Read

Scatttering Reads(分散读取)：如图所示，从一个Channel中读取数据，写入多个Buffer的操作。    

![FVnYwD.png](https://s1.ax1x.com/2018/11/28/FVnYwD.png)  

以下代码演示如何执行Scattering Reads：  

```java
public void scatteringRead() throws Exception {
    try (
        // 创建文件通道
        FileChannel readChannel = new RandomAccessFile("/home/cloudsen/work/java/learning/java11-review/src/umbrella/nio/nio-data.txt",
                                                       "rw").getChannel()
    ) {
        // 创建两个字节缓存区
        ByteBuffer buf1 = ByteBuffer.allocate(5);
        ByteBuffer buf2 = ByteBuffer.allocate(5);
        ByteBuffer bufArray[] = {buf1, buf2};

        // Scattering Reads
        readChannel.read(bufArray);
        printBuffer(buf1);
        printBuffer(buf2);

    }
}

private void printBuffer(ByteBuffer byteBuffer) {
    StringBuilder stringBuilder = new StringBuilder();
    byteBuffer.flip();
    while (byteBuffer.hasRemaining()) {
        stringBuilder.append((char) byteBuffer.get());
    }
    System.out.println(stringBuilder.toString());
}
```

Channel的read()方法传入了一个Buffer数组，然后read()方法按照数组中Buffer对象的顺序，依次给Buffer写入数据，当前的Buffer写满数据后，转到下一个Buffer继续写数据。**这意味着Scattering Read不适合传输动态长度的数据，因为Buffer对象的大小是固定的，而它又是写满一个Buffer才转到下一个Buffer，若消息长度不固定，那么写入Buffer中的数据就是错误的。  **



# Gathering Writes

Gather Write(聚集写入)：如图所示，从多个Buffer中读取数据，写入一个Channel中的操作。  

![FVnRYj.png](https://s1.ax1x.com/2018/11/28/FVnRYj.png)  

以下代码演示如何执行Gathering Writes：  

```java
public void gatheringWrites() throws Exception {
    try (
        // 创建文件通道
        FileChannel writeChannel = new RandomAccessFile("/home/cloudsen/work/java/learning/java11-review/src/umbrella/nio/nio-data-w.txt",
                                                        "rw").getChannel()
    ) {
        // 创建两个字节缓冲区
        ByteBuffer buf1 = ByteBuffer.allocate(2);
        ByteBuffer buf2 = ByteBuffer.allocate(2);
        ByteBuffer bufArray[] = {buf1, buf2};

        String strArray[] = {"a", "b", "c", "d"};
        //写简单数据到缓冲区
        for (int i = 0; i < strArray.length; i++) {
            if (i < 2) {
                buf1.put(strArray[i].getBytes());
            } else {
                buf2.put(strArray[i].getBytes());
            }
        }

        // Gathering Writes
        // 不使用flip()不会写入任何数据
        buf1.flip();
        buf2.flip();
        writeChannel.write(bufArray);
    }
}
```

Channel的write()方法传入了一个Buffer数组，然后write()方法按照数组中Buffer对象的顺序，依次从Buffer中读取数据，写入文件，当前的Buffer数据读完后，转到下一个Buffer继续读数据。**Gathering Writes仅仅写入position和limit之间的数据，并且适用于动态数据。**  



# 参考资料

1. [Java NIO Tutorial - Scatter&Gather](http://tutorials.jenkov.com/java-nio/scatter-gather.html)

