[TOC]

# NIO Buffer简介

Java NIO Buffer用来与NIO Channel进行交互使用。正如你知道的那样，通过channels读取数据写入buffers，或将数据从buffers读取到channels。  

Buffer其实是一个可以写入或读取数据的内存块。对内存块的各种操作被包装在了NIO Buffer对象中。  



# NIO Buffer实现类

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



# NIO Buffer原理

为了理解Buffer的原理，你需要收悉以下3个重要的属性：  

1. position
2. limit
3. capacity

`position` 和 `limit` 的含义取决于缓冲区是处于读取模式还是写入模式。但无论处于什么缓冲模式，`capacity` 的含义不变。  

![FVP6PK.png](https://s1.ax1x.com/2018/11/27/FVP6PK.png)  

## Capacity容量

作为内存块，一个Buffer拥有固定的大小，称为 `capacity` 。您只能将字节，长整数，字符等写入Buffer，一旦Buffer装满了，你就需要清空它，然后才能写入新数据。  

## Position位置

每个数据都在一个确切的 `position` 位置上。初始position为0，当写入数据时，position指向缓冲区中的下一个单元格以插入数据。当position移动到capacity位置时，position为-1。  

## Limit限制

在写模式下，`limit` 和 `capacity` 相等，指你能在缓冲区写入多少数据。将写模式转换为读模式时， `limit` 移动到当前 `position` 的位置，然后position清零。因此在读模式下，limit指可以从数据中读取的最大数据量。  

## Mark标记

`mark` 是一个被记住的位置。调用buffer对象的mark()方法可以设置mark指向当前position。调用reset()方法可以设置position重新指向mark。在设置之前，mark是未定义的。



# NIO Buffer基础用法

使用Buffer写入或读取数据，通常遵循以下5步：  

1. 调用XXXBuffer.allocate()方法给Buffer分配内存空间；
2. 将数据写入Buffer；
3. 调用buffer.flip()方法；
4. 将数据读出Buffer；
5. 调用buffer.clear()或者buffer.compact()方法。

当你写入数据到buffer时，buffer跟踪了你写入的数据量。一旦你需要读取数据，则需要通过flip()方法，将写模式转换到读模式，在读模式状态下，buffer可以读取所有已写入的数据。  

一旦你读取了所有的数据，你可能需要清除buffer，以便再次写入数据。你可以通过两种方式实现这一点：调用flip()或者compact()方法。**clear()方法清除整个缓冲区**。而**compact()方法仅仅清除你已经读取的数据，任何未读数据都会移动到缓冲区的开头，新数据将在未读数据之后写入缓冲区**。  

## 创建Buffer

为了得到一个Buffer对象，你需要分配内存空间，每种Buffer类都有 `allocate()` 方法来实现这一点。如下所示：  

```java
// 创建capacity为48字节大小的字节缓冲区
ByteBuffer bb = ByteBuffer.allocate(48);
// 创建capacity为1024字符大小的字符缓冲区
CharBuffer cb = CharBuffer.allocate(1024);
```

## 写入数据到Buffer

你可以通过两种方式写入数据到Buffer：

1. 将 `Channel` 中的数据写入Buffer；
2. 手动通过Buffer对象的 `put()` 方法直接写入数据。

如下所示：  

```java
// 创建文件通道和字节缓冲区
FileChannel fc = new RandomAccessFile("/xxx/data.txt", "rw").getChannel();
ByteBuffer bb = ByteBuffer.allocate(48);
//通过文件通道读取数据，写入到字节缓冲区
int bytesRead = fc.read(bb);

//手动写入数据
bb.put(3)
```

当然，这里有许多重载的 `put()` 方法允许你使用不同的方式手动写入数据。例如，在某个指定的position处写入数据，或将一个字节数组写入。

## flip()模式转换

通过Buffer对象的 `flip()` 方法，可以将Buffer的写模式，转换为读模式。当flip()方法被调用时，`limit`的值被设置为当前 `position` 的值，然后 `position` 清零。换句话说，position指向读取的位置，limit标记了能够读取的数据量有多少。  

## 从Buffer中读取数据

和写入数据一样，读取数据也有两种方式：  

1. 将Buffer中的数据读取到Channel中；
2. 手动通过Buffer对象的 `get()` 方法直接读出数据。

如下所示：  

```java
FileChannel readChannel = new RandomAccessFile("/xxx/data0.txt", "rw").getChannel();
FileChannel writeChannel = new RandomAccessFile("/xxx/data1.txt", "rw").getChannel();
ByteBuffer bb = ByteBuffer.allocate(48);
int bytesRead = readChannel.read(bb);
// 通过文件通道，读取缓冲区的数据，写入文件
int bytesWrite = writeChannel.write(bb);
// 直接读取缓冲区的数据
bb.get()
```

同样的，这里有许多重载的 `get()` 方法。  

## rewind()

Buffer对象的 `rewind()` 方法，会将position清零，但是limit保持不变。作用就是可以让我们多次读取缓存区的数据。  

```java
buf.flip();
    // 读取buffer数据，通过文件通道将数据写入文件
    writeChannel.write(buf);
    // 将postion清零，limit不变
	// 不清零postion，是无法再次读取的
    buf.rewind();
    // 直接从buffer中读取数据，并打印
    while (buf.hasRemaining()) {
        System.out.println((char) buf.get());
}
```

## clear()和compact()

一旦数据读取完毕(position和limit处于同一位置)，你需要通过 `clear()` 或 `compact()` 让buffer再次准备写入数据。  

`clear()` - **将position清零，limit回到capacity**。注意，整个Buffer被重置了，但是里面的数据没有被清除，再次写入新数据时将覆盖旧数据。当你还有未读数据存在时调用clear()方法，未读的数据将被“遗忘”，因为没有标志告诉你，哪些数据已经读取了，哪些还没有读取。  

`compact()` - **复制所有未读数据到Buffer的开头，然后再将position指向最后一个未读元素之后，limit回到capacity**。  

因此，如果Buffer中存在未读数据，又需要再未读数据后面追加新数据，那么就使用 `compact()` 方法而不是 `clear()` 方法。  

## mark()和reset()

Buffer的 `mark()` 方法可以标记当前position，`reset()` 方法后能将position重置回之前标记的地方。  

```java
buffer.mark();
buffer.get();
buffer.reset();
```

## equals()

可以通过 `equals()` 和 `compareTo()` 来比较两个Buffer对象。  

源码：  

```java
public boolean equals(Object ob) {
    if (this == ob)
        return true;
    if (!(ob instanceof ByteBuffer))
        return false;
    ByteBuffer that = (ByteBuffer)ob;
    if (this.remaining() != that.remaining())
        return false;
    return BufferMismatch.mismatch(this, this.position(),
                                   that, that.position(),
                                   this.remaining()) < 0;
}

static int mismatch(ByteBuffer a, int aOff, ByteBuffer b, int bOff, int length) {
    int i = 0;
    if (length > 7) {
        // 这里用get()比较每个剩余元素
        if (a.get(aOff) != b.get(bOff))
            return 0;
        i = ArraysSupport.vectorizedMismatch(
            a.base(), a.address + aOff,
            b.base(), b.address + bOff,
            length,
            ArraysSupport.LOG2_ARRAY_BYTE_INDEX_SCALE);
        if (i >= 0) return i;
        i = length - ~i;
    }
    for (; i < length; i++) {
        if (a.get(aOff + i) != b.get(bOff + i))
            return i;
    }
    return -1;
}
```

**equals仅比较Buffer的剩余元素，而不是它内部的每个元素，如果每个缓冲区的剩余内容相同，则equals()方法返回true;否则，它返回false。**   

剩余内容满足以下所有条件，则相等：  

1. 都是相同的类型。即包含不同数据类型的缓冲区永远不会相等，并且Buffer不可能等于非Buffer对象；
2. position到limit之间具有相同数量的剩余元素。注意，Buffer的capacity不必相同，在Buffer中的剩余元素的index也不必相同；
3. 从get()返回的剩余数据元素序列在每个缓冲区中必须相同。  

```java
 public void testBufferEquals() throws Exception {
     try (
         // 创建文件通道
         FileChannel readChannel = new RandomAccessFile("/home/cloudsen/work/java/learning/java11-review/src/umbrella/nio/nio-data.txt",
                                                        "rw").getChannel();
         FileChannel readChannel2 = new RandomAccessFile("/home/cloudsen/work/java/learning/java11-review/src/umbrella/nio/nio-data.txt",
                                                         "rw").getChannel();
         FileChannel readChannel3 = new RandomAccessFile("/home/cloudsen/work/java/learning/java11-review/src/umbrella/nio/nio-data.txt",
                                                         "rw").getChannel();
         FileChannel readChannel4 = new RandomAccessFile("/home/cloudsen/work/java/learning/java11-review/src/umbrella/nio/nio-data.txt",
                                                         "rw").getChannel()
     ) {
         // 创建三个内存空间大小为5的字节缓存区
         ByteBuffer buf5 = ByteBuffer.allocate(5);
         ByteBuffer buf5a = ByteBuffer.allocate(5);
         ByteBuffer buf5b = ByteBuffer.allocate(5);
         // 创建内存空间大小为10的字节缓存区
         ByteBuffer buf10 = ByteBuffer.allocate(10);

         System.out.println("=".repeat(20));
         // true: position到limit之间元素数量相同
         System.out.println("buf5 equals to buf5a? " + buf5.equals(buf5a));
         // false：position到limit之间元素数量不相同
         System.out.println("buf5 equals to buf10? " + buf5.equals(buf10));
         // false
         System.out.println("buf5a equals to buf10? " + buf5a.equals(buf10));

         // 使用相同的通道，装满buffer
         readChannel.read(buf5);
         readChannel.read(buf5a);
         readChannel.read(buf10);

         // 读取数据后，position和limit在同一元素，没有剩余元素。
         System.out.println("=".repeat(20));
         // true
         System.out.println("buf5 equals to buf5a? " + buf5.equals(buf5a));
         // true
         System.out.println("buf5 equals to buf10? " + buf5.equals(buf10));
         // true
         System.out.println("buf5a equals to buf10? " + buf5a.equals(buf10));

         /*
            buf5 content:
            abcde
            buf5a content:
            fghij
            buf10 content:
            klmnopqrst
             */
         System.out.println("=".repeat(20));
         System.out.println("buf5 content:");
         this.printBuffer(buf5);
         System.out.println("buf5a content:");
         this.printBuffer(buf5a);
         System.out.println("buf10 content:");
         this.printBuffer(buf10);

         // 重置position
         buf5.rewind();
         buf5a.rewind();
         buf10.rewind();

         System.out.println("=".repeat(20));
         // false 剩余元素数量相同但是内容不同
         System.out.println("buf5 equals to buf5a? " + buf5.equals(buf5a));
         // false 剩余元素数量不相同
         System.out.println("buf5 equals to buf10? " + buf5.equals(buf10));
         // false 剩余元素数量不相同
         System.out.println("buf5a equals to buf10? " + buf5a.equals(buf10));

         // 清空buffer，通过不同channel读取相同的内容
         buf5.clear();
         buf5a.clear();
         buf10.clear();
         readChannel2.read(buf5);
         readChannel3.read(buf5a);
         readChannel3.read(buf5b);
         readChannel4.read(buf10);

         /*
            buf5 content:
            abcde
            buf5a content:
            abcde
            buf5b content:
            fghij
            buf10 content:
            abcdefghij
             */
         System.out.println("=".repeat(20));
         System.out.println("buf5 content:");
         this.printBuffer(buf5);
         System.out.println("buf5a content:");
         this.printBuffer(buf5a);
         System.out.println("buf5b content:");
         this.printBuffer(buf5b);
         System.out.println("buf10 content:");
         this.printBuffer(buf10);


         // 重置position
         buf5.rewind();
         buf5a.rewind();
         buf10.rewind();

         System.out.println("=".repeat(20));
         // true 剩余元素数量相同, 且内容也相同都是abcde
         System.out.println("buf5 equals to buf5a? " + buf5.equals(buf5a));
         // false 剩余元素数量不相同
         System.out.println("buf5 equals to buf10? " + buf5.equals(buf10));
         // false 剩余元素数量相同, 但内容不同，一个是abcde一个是fghij
         System.out.println("buf5a equals to buf5b? " + buf5a.equals(buf5b));
         // false 剩余元素数量不相同
         System.out.println("buf5a equals to buf10? " + buf5a.equals(buf10));
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

## compareTo()

看看Buffer的定义，它实现了Comparable接口：  

```java
public abstract class XXXBuffer
    extends Buffer
    implements Comparable<XXXBuffer>{
    //...
}
```

Buffer对象的compareTo()方法也是用于比较两个Buffer的剩余元素。此方法返回负，零或正整数，对应小于，等于或大于。在排序程序中也许会用到。  

若满足以下条件，则缓冲区被视为“小于”另一个缓冲区：  

1. 第一个元素小于另一个缓冲区的对应元素(字典比较)；
2. 缓冲区的元素数量小于另一个缓冲区。

注意，因为实现了Comparable接口，这意味着可以通过调用java.util.Arrays.sort()方法根据内容对Buffer数组进行排序。  



# 参考资料

1. [Java NIO Tutorial - Buffer](http://tutorials.jenkov.com/java-nio/buffers.html)
2. [How To Do In Java - Buffer](https://howtodoinjava.com/java7/nio/java-nio-2-0-working-with-buffers/)

