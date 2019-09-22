[TOC]

# RabbitMQ核心概念

RabbitMQ是一个基于AMQP协议实现的消息Broker，你可以把它类比为邮箱、邮局、邮递员组成的邮政系统，但不同的地方是RabbitMQ只接收、存储和转发消息。  

使用MQ的优点是：  

- 异步化耗时操作；
- 使应用松耦合；
- 增加应用的可扩展性；
- 高峰期时削峰；

## AMQP

Advanced Message Queuing Protocol(AMQP)即高级消息队列协议，是用于统一消息中间件实现的一套标准。它专注与各种消息的传递，因此可以用于不同语言编写的应用间的消息传递。  

RabbitMQ基于[AMQP 0-9-1](https://www.rabbitmq.com/amqp-0-9-1-quickref.html)实现，可以通过插件的方式升级为1.0协议。  

简单来说，AMQP规定了消息需要使用 `Producer`，`Consumer` 和 `Broker` 。  

## Broker

消息服务器，消息协调器，可以看作rabbitmq-server，提供了消息核心服务。  

## VHOST

虚拟主机，一个broker里可以开设多个vhost，用作不同用户的权限分离。  

## Producer

消息生产者，业务的发起方，负责发送消息给Broker中的Exchange。  

## Consumer

消息消费者，业务的处理方，负责从Broker中的Exchange获取消息，并进行处理。  
## Queue

队列，存放一堆待消费的消息。  

## Message

消息体，封装了业务需要的数据，用于消息传输。  

## Exchange

交换器，会与Queue进行绑定，它和Queue是多对多的关系。  

它将 `routing key` 与 `binding key` 相比较，然后将消息分发给对应的 `queue`。消息的分发方式取决于Exchange的类型。  

它有以下几种类型：  

- **nameless**： 默认，将收到的消息发送给 `routing key` 与队列名匹配的队列；

- **fanout**：将收到的消息直接发给所有与之绑定的队列；
- **direct**：将收到的消息发送给 `routing key` 与 `binding key` 匹配的队列；
- **topic**：routing key可以模式匹配；
- **headers**: 【不推荐】性能差，通过message的消息头来匹配queue；

## Binding

绑定，指Exchange和Queue之间的绑定，通过Binding Key来完成。  

## Binding Key

绑定键，其实就是指的 `routing key`，Exchange和Queue绑定的时候一般说绑定键。

## Routing Key

路由键，exchange根据这个键值筛选队列。  

## Channel

消息通道，客户端与Broker通过通道建立连接，多路复用。  

在客户端的每个连接里，可建立多个channel，每个channel代表一个会话任务。



# RabbitMQ结构图

了解以上概念后，再看看这张图片，什么都清楚了：  

![rabbit](https://s2.ax1x.com/2019/09/22/upOFUK.png)  









