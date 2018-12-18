[TOC]

# Java NIO Selector简介

`Selector` 是用来监控一个或多个NIO Channel的组件，它在监控的同时，还决定哪些Channels准备好读写数据。使用 `Selector` 后，一个单线程能够管理多个Channel，从而能够管理多个网络连接。  

![FEp3wV.png](https://s1.ax1x.com/2018/11/27/FEp3wV.png)  



# 为什么使用Selector

> 请记住，现代操作系统和CPU在多任务处理方面变得越来越强大，因此多线程的开销随着时间的推移变得越来越小，现在20几核的CPU也是便宜得不行。如果CPU有多个核心，而你不使用多任务，那就是在浪费CPU资源。

Selector可以让你使用更少的线程来管理Channels。事实上，你可以仅使用一个线程来管理所有的Channels。对于操作系统而言，线程之间频繁切换是十分占用资源的，并且每个线程都占用了相应的内存资源。因此，你的程序占用的线程越少越好。  

  

# 创建Selector

可以调用Selector类的 `open()` 方法来生成一个selector对象：  

```java
import java.nio.channels.Selector;
Selector selector = Selector.open();
```



# 给Selector注册Channels

要使用Selector的Channels，必须先进行注册。只有满足以下两点的Channels才能与Selector一起使用：  

1. **Channels必须是 `SelectableChannel` 的子类。**
2. **Channels必须处于 `non-blocking mode` 非阻塞模式。**

因此 `FileChannel` 就不能与Selector一起使用了，因为它不能转换到非阻塞模式。然而 Socket channels却可以正常工作。  

将一个Channel注册到Selector，使用的是SelectableChannel类的 `register()` 方法，这个方法接收以下三个参数：  

1. `sel` - 注册此Channel的Selector；

2. `ops` - “interest set”监听集，设置Selector监听的Channel事件；

3. `att` - 最终结果SelectionKey的附属对象，可为null。

`register()` 方法将返回一个SelectionKey，这个key就代表了本次注册。

对于第二个参数，可以选择以下四种事件进行监听：  

1. Connect连接；
2. Accept接收；
3. Read读取；
4. Write写入。

我们可以将“Channel触发某个事件”称为“Channel的某个事件就绪”。那么，当Channel成功连接服务器时，叫做“connect ready”；当SocketChannel收到某个请求时，叫做“accept ready”；当一个Channel已经准备好数据，需要被读取时，叫做“read ready”；当一个Channel需要你写入数据时，叫做“write ready”。  

以上的这四种事件，已经由 `SelectionKey` 类包装为常量了，我们可以通过它直接得到：  

1. SelectionKey.OP_CONNECT == 1 == 0000 0001
2. SelectionKey.OP_ACCEPT == 4 == 0000 0100
3. SelectionKey.OP_READ == 8 == 0000 1000
4. SelectionKey.OP_WRITE == 16 == 0001 0000

一个简单的注册大概是这样的：  

```java
//...
// 开启non-blocking模式
channel.configureBlocking(false);
// 将该通道注册到对应的选择器，让选择器监听Read Ready
SelectionKey key = channel.register(selector, SelectionKey.OP_READ);
//...
```

如果你想监听多个事件，可以把事件常量进行 `按位或` 操作，就像下面这样：  

```java
int interestSet = SelectionKey.OP_CONNECT | SelectionKey.OP_READ;
/*
OP_CONNECT 0000 0001
OP_READ    0000 1000
		|
		=  0000 1001
*/
```

这个后面再详细讨论。  



# SelectionKey

正如上面提到的，当你使用了 `register()` 方法注册Channel到某个Selector时，它会返回一个 `SelectionKey` 。它由以下内容构成：  

1. “interest set”监听集，希望Selector监听的事件；
2. “ready set”就绪集；
3. 注册的Channel；
4. 目标Selector；
5. 一个附带的对象（可选的）。

## Interest Set

“interest set”监听集，是你希望Selector监听的事件集。你可以通过 `SelectionKey` 对象的 `interestOps()` 方法来得到监听集：  

```java
int interestSet = selectionKey.interestOps();
boolean isInterestedInAccept  = (interestSet & SelectionKey.OP_ACCEPT) == SelectionKey.OP_ACCEPT;
boolean isInterestedInConnect = (interestSet & SelectionKey.OP_CONNECT) == SelectionKey.OP_CONNECT;
boolean isInterestedInRead    = (interestSet & SelectionKey.OP_READ) == SelectionKey.OP_READ;
boolean isInterestedInWrite   = (interestSet & SelectionKey.OP_WRITE) == SelectionKey.OP_WRITE;

/*
eg:
OP_CONNECT|OP_READ 0000 1001
OP_CONNECT         0000 0001
				&
				=  0000 0001	true

OP_CONNECT|OP_READ 0000 1001
OP_WRITE           0001 0000
				&
				=  0000 0000	false
*/
```

如上所示，你可以通过 `按位与` 操作，来判断某个事件是否被设置到了监听集中。  

## Ready Set

“ready set”就绪集，是Channel已经就绪的操作集。当Selector选择了某个Channel后，就会访问这个就绪集，读取相应操作的就绪状态。你可以通过 `SelectionKey` 对象的 `readyOps()` 来访问它：  

```java
int readySet = selectionKey.readyOps();
```

当然，你也可以像监听集那样，通过 `按位与` 来判断某个操作是否就绪。但是，`SelectionKey` 对象为我们提供了更方便的方法：  

```java
// 它们都返回Boolean
selectionKey.isAcceptable();
selectionKey.isConnectable();
selectionKey.isReadable();
selectionKey.isWritable();
```

## 注册的Channel和目标Selector

可以通过SelectionKey得到Channel和关联的Selector：  

```java
Channel  channel  = selectionKey.channel();
Selector selector = selectionKey.selector();
```

## 附加对象

您可以将对象附加到SelectionKey，这样一来，可以通过附加对象识别给定Channel或将更多信息附加到Channel。例如，你可以附带一个Channel用到的Buffer对象或是一个聚合多种数据的对象。通过 `attach()` 方法或 `channel.register()` 方法可以附加一个对象，通过 `attachment()` 方法可以得到附加对象。 

```java
// 添加附加对象
SelectionKey key = channel.register(selector, SelectionKey.OP_READ, theObject);
selectionKey.attach(theObject);
// 获得附加对象
Object attachedObj = selectionKey.attachment();
```



# 通过Selector选择Channel

一旦Selector注册了一个或多个Channel，就可以调用各种 `select` 方法。这个方法会返回监听事件已就绪的Channel。比如，返回一个读取就绪“read ready”的Channel。  

以下是常用的 `select` 方法：  

- int select()
- int select(long timeout)
- int selectNow()

**select()** - 一直阻塞(blocking)，直到注册的Channels中，至少有一个Channel的监听事件就绪，或被 `wakeup` 或被 `interrupt`。  

**select(long timeout)** - 与select()相同，但是有一个超时时间（单位是毫秒）。超时之后，也会返回。  

**selectNow()** - 使用non-blocking非阻塞，它会立即返回监听事件就绪的任何一个Channel，如果找不到适合的，就立即返回0。  

以上的select方法都会返回一个整数，表示自上次调用select()以来，有多少个就绪的Channels。  

## selectedKeys()

通过 `channel.register()` 方法把一个Channel注册到某个Selector时，返回了一个 `SelectionKey` ，它表示了Channel和Selector的关系。一旦你调用了Selector的select()方法，它就会返回整数，以此表示一个或多个Channel已经就绪，这个返回的整数，其实是指的SelectionKey的数量。你可以通过Selector的 `selectedKeys()` 方法来访问已就绪的SelectionKey集合(selected-key set)，这个集合很形象的称作“已选键集合”。  

```java
Set<SelectionKey> selectedKeys = selector.selectedKeys();
```

然后遍历这个selected-key集合，通过每个SelectionKey就能得到对应的就绪状态和Channel：  

```java
Set<SelectionKey> selectedKeys = selector.selectedKeys();

Iterator<SelectionKey> keyIterator = selectedKeys.iterator();

while(keyIterator.hasNext()) {
    
    SelectionKey key = keyIterator.next();

    if(key.isAcceptable()) {
        // 连接已被ServerSocketChannel接收
        Channel  channel = key.channel();
		// ...
    } else if (key.isConnectable()) {
        // 已与远程服务器建立连接

    } else if (key.isReadable()) {
        // 通道已准备好被读取数据

    } else if (key.isWritable()) {
        // 通道已准备好被写入数据
    }

    keyIterator.remove();
}
```

上面的简单示例通过keyIterator.remove()，手动移除了selected-key集合中已处理的SelectionKey。这是因为Selector不会自动的去删除它们，每当处理完毕一个事件时，手动删除，等到下次事件再次就绪时，对应的SelectionKey又会自动加入selected-key集合。  

SelectionKey.channel()应该被转换为你需要的特定Channel，如ServerSocketChannel或SocketChannel 等等。  



# wakeup()

当线程因为调用select()方法而被阻塞时，可以在另外的线程中使用Selector的 `wakeup()` 方法，即使没有就绪的Channel，其他线程也能从select()中返回。  

如果有其它线程调用了wakeup()方法，但当前没有线程阻塞在select()方法里，那么下个调用select()方法的线程会立即wake up并返回。  



# close()

`close()` 方法能关闭某个Selector。如果调用close()时，当前线程在select()方法中阻塞，则会直接中断并关闭。  

当关闭Selector时，任何与它关联的SelectionKey都会失效，SelectionKey对应的Channels也被注销，然后其他任何有关联的资源都会被释放。Channels本身不会被关闭。



# 简单应用

```java
Selector selector = Selector.open();

channel.configureBlocking(false);

SelectionKey key = channel.register(selector, SelectionKey.OP_READ);


while(true) {

  int readyChannels = selector.select();

  if(readyChannels == 0) continue;


  Set<SelectionKey> selectedKeys = selector.selectedKeys();

  Iterator<SelectionKey> keyIterator = selectedKeys.iterator();

  while(keyIterator.hasNext()) {

    SelectionKey key = keyIterator.next();

    if(key.isAcceptable()) {
        // a connection was accepted by a ServerSocketChannel.

    } else if (key.isConnectable()) {
        // a connection was established with a remote server.

    } else if (key.isReadable()) {
        // a channel is ready for reading

    } else if (key.isWritable()) {
        // a channel is ready for writing
    }

    keyIterator.remove();
  }
}
```





