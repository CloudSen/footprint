# Nginx指南 #2基本概念



## 什么是Nginx？

> C10K：web2.0初期，不能很好地去解决每秒上万次的并发请求，只有靠堆硬件勉强硬撑。现在面对的问题是C10M，每秒千万级别的并发请求。
>
> 上游服务器(upstream servers)：上游，有源头的意思，即代表tomcat、uWSGI之类的产生内容的服务器。

Nginx最初是为了解决[C10K问题](https://en.wikipedia.org/wiki/C10k_problem)而创造的产物。首先它可以为一个WEB服务器，快速地为你的数据服务。其次你可以使用它实现反向代理(reverse proxy)，并轻易的集成一些如`Unicorn` 和 `Puma` 这样相对缓慢的上游服务器。当然，你还可以用它实现负载均衡(load balancing)，合理分流、动态调整图片、内容缓存、IMAP、POP3、SMTP等等。

Nginx最基本的结构是由master进程和多个相互独立的worker进程组成。master会读取配置文件以及管理workers，workers会处理requests请求。  

  

> 版权信息
>
> 本文大量参考他人成果以及官方文档，本人进行了翻译和归纳总结，原作者信息如下：
>
> > 作者：Mateusz Dobek
> >
> > 链接：[Nginx Tutorial #1: Basic Concepts](https://www.netguru.co/codestories/nginx-tutorial-basics-concepts)
> >
> > 发布于：netguru