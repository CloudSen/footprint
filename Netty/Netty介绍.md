# Netty 介绍

Netty是一个NIO客户端服务器框架，它可以快速、轻松地开发协议服务器和客户端等网络应用程序。它大大简化和简化了网络编程，如TCP和UDP套接字服务器。  

“快速简便”并不意味着最终的应用程序将遭受可维护性或性能问题的困扰。 Netty经过精心设计，结合了许多协议（例如FTP，SMTP，HTTP以及各种基于二进制和文本的旧式协议）的实施经验。 因此，Netty成功地找到了一种无需妥协即可实现轻松开发，高性能，高稳定性和灵活性的方法。  

## 特性

### 设计

- 各种传输类型的统一API（阻塞和非阻塞套接字）
- 基于灵活和可扩展的事件模型，允许明确的分离业务
- 高度可定制的线程模型——单个线程，一个或多个线程池，如SEDA
- 真正无需连接的数据报文套接字支持

### 简单易用

- 有丰富的Java文档，用户指南和代码案例
- 没有额外的依赖，JDK 5 (Netty 3.x)或6 (Netty 4.x)就足够
  - 注意：一些例如HTTP/2的组件可能有额外的依赖，更多请查看 [依赖说明页面](https://netty.io/wiki/requirements.html)。

### 性能

- 更强的吞吐量以及低延迟
- 更少的资源消耗
- 最小化不必要的内存副本

### 安全

- 完全支持 `SSL/TLS` 和 `StartTLS` 协议

