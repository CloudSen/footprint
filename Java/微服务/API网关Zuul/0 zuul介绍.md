# Zuul介绍

路由是微服务架构中不可缺失的一部分。例如，`/` 路径会映射到你的web应用根目录，`/api/user` 会映射到用户微服务，`/api/shop` 会映射到购物微服务。`Zuul` 是Netflix提供的基于JVM的路由器、服务端负载均衡器。  

我们可以通过 `Zuul` 做以下事情：  

- Authentication 权限认证
- Insights 洞察
- Stress Testing 压力测试
- Canary Testing 金丝雀测试
- Dynamic Routing 动态路由
- Service Migration 服务迁移
- Load Shedding 减载
- Security 安全
- Static Response handling 处理静态响应
- Active/Active traffic management 流量管理

`Zuul` 的规则引擎能够让编写的规则和过滤器写进几乎所有的JVM语言，内建的支持JAVA和Groovy语言。  

> 旧的 `zuul.max.host.connections` 配置属性，已经被 `zuul.host.maxTotalConnections` 和 `zuul.host.maxPerRouteConnections` 配置属性替代，它们默认值分别是200和20。  

> `Hystrix` 对所有路由默认的 `ExecutionIsolationStrategy` 隔离模式是 `SEMAPHORE`。Zuul可以通过 `zuul.ribbonIsolationStrategy` 设置为 `THREAD` 来强制使用线程模式。

