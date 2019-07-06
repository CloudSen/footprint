# Turbine Stream

在某些环境中（如，PaaS），Turbine拉取所有Hystrix commands的metrics指标可能失效。在这种情况下，我们希望Hystrix commands自己push指标到Turbine。Spring Cloud通过消息来实现这个功能。  

在客户端，需要加入 `spring-cloud-netflix-hystrix-stream` 和你所选择的 `spring-cloud-starter-stream-*` 依赖，更多详情见[Spring Cloud Stream](https://docs.spring.io/spring-cloud-stream/docs/current/reference/htmlsingle/)。

在Turbine Stream服务端，因为需要使用 `Spring Webflux` ，因此需要加入 `spring-boot-starter-webflux` 依赖。默认情况下， `spring-cloud-starter-netflix-turbine-stream` 包含了 `spring-boot-starter-webflux` ，因此引入它就足够了。然后可以根据加入 Stream Binder，如 `spring-cloud-starter-stream-rabbit` 。  

依赖引入完毕后，就可以Hystrix Dashboard指向Turbine Stream服务端而不是单个的Hystrix streams。比如，如果Turbine Stream服务运行在8989端口，则你需要将 `http://yourHost:8989` 填入Hystrix Dashboard中。断路器会以这种方式命名：`{serviceId}.{断路器名字}` 。  

Turbine Steam服务端的URL同样支持 `cluster` 参数。它不像一般的Turbine服务，Turbine Stream只能通过Eureka的 `serviceId` （小写）作为集群名，这是不可配置的，你不需要 `turbine.appConfig` 之类的配置。  

如果Turbine Steam服务端运行在 `you.host:8989`，并且拥有两个Eureka服务 `customers` 和 `products` 。  

下面的URLs都是可用的，`default` 和空的集群名会展示所有的指标（***注意集群名这里是小写***）：  

```
https://my.turbine.sever:8989/turbine.stream?cluster=customers
https://my.turbine.sever:8989/turbine.stream?cluster=products
https://my.turbine.sever:8989/turbine.stream?cluster=default
https://my.turbine.sever:8989/turbine.stream
```

