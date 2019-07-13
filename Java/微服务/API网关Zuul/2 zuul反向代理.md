# Zuul内置的反向代理

## 介绍

Spring Cloud为前后端分离项目提供了内置的Zuul代理，方便前端应用调用一个或多个后端服务，避免了为每个后端服务的CORS和权限认证进行管理。  

在Zuul服务端的主启动类上添加 `@EnableZuulProxy` 即可启用该功能。这样一来，本地调用就会转发到相应的应用服务中去。例如，一个ID为 `users` 的微服务会收到 `/users` 代理中的请求（微服务中收到的是删除 `/users` 前缀的请求）。代理通过Ribbon来定位服务实例。所有的请求都在 `Hystrix command` 断路器中被执行，因此失败的请求会显示在 `Hystrix metrics `中，一旦断路器被开启，当前代理就不会再尝试连接对应的微服务。  

> Zuul的starter没有包含服务发现客户端，因此如果你需要通过微服务的ID来进行路由，手动加入POM依赖即可。



## 配置路由

可以通过配置文件直接配置代理路由：  

***application.yml***  

```yaml
zuul:
  routes:
    users: /myusers/**
```

上面的例子中，如果有一个 `/myusers` 相关的HTTP请求，就会被转发到 `users` 微服务中处理，并且默认情况下会删除前缀。如，`/myusers/101` 请求会转发到 `users` 微服务的 `/101` 路由。  

为了获得更加颗粒度的配置，你可以单独指定 `path` 和 `serviceId` ：  

```yaml
zuul:
  routes:
    users:
      path: /myshop/**
      serviceId: myshop_service
```

上面的例子中，如果有一个 `/myshop` 相关的HTTP请求，就会被转发到 `myshop_service` 微服务中处理。  

路由必须有一个 `path` ，它可以使用 `ant-style` 模式匹配，因此 `/myshop/*` 只会匹配一级路径，而 `/myshop/**` 才会匹配多层级的路径。    

如果不希望请求在转发前删除前缀可以通过以下方式配置：  

```yaml
zuul:
  routes:
    users:
      path: /myusers/**
      stripPrefix: false
```

上面的例子中，`/myusers/101` 请求会转发到 `users` 微服务的 `/myusers/101` 路由。

如果想为所有 `path` 添加统一的前缀，可以这样配置：  

```yaml
zuul:
  prefix: /api
  routes:
    users:
      path: /myusers/**
```

定位微服务的方式有两种，通过指定 `serviceId` （服务发现中的名字）或者通过指定 `url` （物理地址）：  

 ```yaml
zuul:
  routes:
    users:
      path: /myusers/**
      url: https://example.com/users_service
 ```

但是，这种简单的URL路由不会以 `HystrixCommand` 的方式执行，也不能设置多个URLs通过Ribbon进行负载均衡。通过下面这种方式可以实现断路器和负载均衡，设置 `serviceId` 存储一个静态的服务列表：  

```yaml
zuul:
  routes:
    echo:
      path: /myusers/**
      serviceId: myusers-service
hystrix:
  command:
    myusers-service:
      execution:
        isolation:
          thread:
            timeoutInMilliseconds: ...
myusers-service:
  ribbon:
    NIWServerListClassName: com.netflix.loadbalancer.ConfigurationBasedServerList
    listOfServers: https://example1.com,https://example2.com,...
    ConnectTimeout: 1000
    ReadTimeOut: 3000
    MaxTotalHttpConnections: 500
    MaxConnectionsPerHost: 100
```

还可以通过另外一种配置方法：  

```yaml
zuul:
  routes:
    path: /myusers/**
    serviceId: users
ribbon:
  eureka:
    enabled: false
users:
  ribbon:
    listOfServers: https://example1.com,https://example2.com,...
```



## 失败重试

`zuul.routes` 条目实际上会绑定到 `ZuulProperties` 类。如果查看该类的属性，可以看到它还具有可重试的标志 `retryable` 。将该标志设置为true便可使Ribbon客户端自动重试失败的请求。  

```yaml
zuul:
  retryable: true
```



##   关于请求头

默认情况下，`X-Forwarded-Host` 会被添加到转发的请求中去，如果你不想这样，可以设置 `zuul.addProxyHeaders=false`。此外，默认情况下 `path` 中的前缀会被删除，并且转发的请求头中会加上 `X-Forwarded-Prefix` 。



## 忽略某个服务

为了防止某个服务自动被添加，可以给 `zuul.ignored-services` 设置一个要忽略的模式匹配列表。如果一个服务的ID匹配其中的某个模式，这个服务就会被忽略。但，如果服务与忽略的模式匹配，同时又包含在路由映射中，该服务不会被忽略。如下所示：  

***application.yml***  

```yaml
zuul:
  ignoredServices: '*'
  routes:
    users: /users/**
```

上面的例子中，因为在路由映射中配置了 `/users/**` ，因此user服务不会被忽略掉，其余的服务都会被忽略。  



## 忽略某个路径映射

为了适应某些复杂的场景，可以给 `zuul.ignoredPatterns` 设置需要忽略的路径映射的模式匹配：  

```yaml
zuul:
  ignoredPatterns: /**/admin/**
  routes:
    users: /myusers/**
```

上面的例子中，只要包含了admin的路径都会被忽略。 



## 路径模式匹配的顺序

只有YAML配置文件才能通过路径写入的顺序进行模式匹配，PROPERTIES文件的路径模式匹配没有顺序！  

```yaml
zuul:
  routes:
    users:
      path: /myusers/**
    legacy:
      path: /**
```

上面的例子中，会先使用users的路径匹配，然后再使用legacy的。如果这里使用PROPERTIES文件，则users路由永远无法使用。  



## 自定义HTTP客户端

由于Ribbon的 `RestClient` 已经过时了，因此，Zuul使用Apache HTTP Client作为默认的HTTP客户端。  

如果你想更换其他的客户端，可以见Ribbon章节。  



## 忽略Headers

有时候我们会对Header比较敏感，可以通过 `zuul.ignoredHeaders` 来全局设置一个Header忽略列表，以此在与下游服务器交互前丢弃这些Header（request和response都会生效）。  

如果想忽略Spring Security的请求头，可以设置 `zuul.ignoreSecurityHeaders=false` 。



## 暴露的端点

默认情况下，当使用了 `@EnableZuulProxy` 注解后，当前应用会暴露两个端点： `/routes` 和 `/filters` 。  

### Routes端点

在端点管理中开启 `routes` 端点：  

```
management:
  endpoints:
    web:
      exposure:
        include: routes
```

通过GET请求访问 `/actuator/routes` 端点，会返回已经映射的路由列表：  

```json
{
    "/api/sys-management/**": "system-management",
    "/api/demo-for-test/**": "demo-for-test"
}
```

访问 `/routes?format=details` 会返回更详细的信息：   

```json
{
  "/stores/**": {
    "id": "stores",
    "fullPath": "/stores/**",
    "location": "http://localhost:8081",
    "path": "/**",
    "prefix": "/stores",
    "retryable": false,
    "customSensitiveHeaders": false,
    "prefixStripped": true
  }
}
```

通过POST请求访问 `/routes` 端点，能刷新已存在的路由。  

### Filters端点

在端点管理中开启 `filters` 端点：  

```json
management:
  endpoints:
    web:
      exposure:
        include: filters
```

通过GET请求访问 `/actuator/filters` 端点，能够返回一个以类型为key的map：  

```json
{
    "error": [
        {
            "class": "org.springframework.cloud.netflix.zuul.filters.post.SendErrorFilter",
            "order": 0,
            "disabled": false,
            "static": true
        }
    ],
    "post": [
        {
            "class": "org.springframework.cloud.netflix.zuul.filters.post.SendResponseFilter",
            "order": 1000,
            "disabled": false,
            "static": true
        }
    ],
    "pre": [
        {
            "class": "org.springframework.cloud.netflix.zuul.filters.pre.DebugFilter",
            "order": 1,
            "disabled": false,
            "static": true
        },
        {
            "class": "org.springframework.cloud.netflix.zuul.filters.pre.FormBodyWrapperFilter",
            "order": -1,
            "disabled": false,
            "static": true
        },
        {
            "class": "org.springframework.cloud.netflix.zuul.filters.pre.Servlet30WrapperFilter",
            "order": -2,
            "disabled": false,
            "static": true
        },
        {
            "class": "org.springframework.cloud.netflix.zuul.filters.pre.ServletDetectionFilter",
            "order": -3,
            "disabled": false,
            "static": true
        },
        {
            "class": "org.springframework.cloud.netflix.zuul.filters.pre.PreDecorationFilter",
            "order": 5,
            "disabled": false,
            "static": true
        }
    ],
    "route": [
        {
            "class": "org.springframework.cloud.netflix.zuul.filters.route.SimpleHostRoutingFilter",
            "order": 100,
            "disabled": false,
            "static": true
        },
        {
            "class": "org.springframework.cloud.netflix.zuul.filters.route.RibbonRoutingFilter",
            "order": 10,
            "disabled": false,
            "static": true
        },
        {
            "class": "org.springframework.cloud.netflix.zuul.filters.route.SendForwardFilter",
            "order": 500,
            "disabled": false,
            "static": true
        }
    ]
}
```



