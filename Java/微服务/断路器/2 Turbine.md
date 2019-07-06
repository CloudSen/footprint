# Turbine 集群监控

`Hystrix Dashboard` 通过 `/hystrix.stream` 监控，只能监控单个节点。  

然而就整个系统的健康状况而言，比如在集群中，监控单个 `Hystrix` 客户端的数据没多少意义。`Turbine` 是一个应用程序，它将所有分散的 `/hystrix.stream` 端点聚合为一个 `/turbine.stream` 流，提供给 `Hystrix Dashboard` 使用，以便集中监控集群的状态。`Turbine` 会通过 `Eureka` 寻找服务实例。  

> 默认情况下，`Turbine` 会在注册的服务实例中，通过 `hostName` 和 `port` 来查找对应的 `/hystrix.stream` 端点，然后将该 `/hystrix.stream` 附加到 `Turbine` 中。  
>
> 如果 `Eureka` `instance` 的 `metadata` 配置中包含了 `management.port` ，`Turbine` 会使用 `management.port` 而不是 `port` 去定位 `/hystrix.stream` 端点。   
>
> 你可以通过这样来自定义：  
>
> ```yaml
> eureka:
>   instance: 
>     metadata-map:
>       management.port: ${management.port:8081}
> ```

## Turbine 配置说明

> 详细的配置见[官网wiki](<https://github.com/Netflix/Turbine/wiki/Configuration-(1.x)>)

- `turbine.instanceUrlSuffix` 配置设置turbine的URL前缀；
- `turbine.instanceInsertPort` 配置接收 `true` 或者 `false` ：默认是 `true`，即 `turbine.instanceUrlSuffix` 内容不需要写 `:port` 前缀；

- `turbine.appConfig` 配置项接收一个列表，存放的是在 `Eureka` 注册的 `serviceIds`( `spring.application.name` 配置指定的)，`Turbine` 通过它来寻找微服务实例，表明监控哪些服务；

- `turbine.aggregator.clusterConfig` 指定要聚合哪些集群：

  ***`Turbine` 流通过一个URL给 `Hystrix Dashboard` 使用：  `https://your.server:port/turbine.stream?cluster=CLUSTERNAME` 。***如果集群的名字是 `default`，则可以忽略 `cluster` 参数。`cluster` 参数必须存在于 `turbine.aggregator.clusterConfig` 配置项中。***注意这里的数据必须是大写的。***因此，以下是一个在Eureka中注册的，名叫 `customers` 的应用对应的Turbine配置：  

  ```yaml
  turbine:
    aggregator:
      cluster-config: CUSTOMERS
    app-config: customers
  ```

  如果你不想在 `turbine.aggregator.clusterConfig` 中存储群集名称，而是希望自定义 `Turbine` 应该使用集群名称，那么你需要提供一个 `TurbineClustersProvider` 类。  

- `turbine.combine-host-port` 接收 `true` 或 `false` ：默认情况下是 `true` ，使用host加port的方式来聚合服务。若设置为 `false`，则仅以host来区分不同的服务，可能会将不同的服务聚合为一个。

- `turbine.cluster-name-expression` 用于自指定集群名称的来源，原理是使用SPEL(Spring Expression Language)表达式从 `InstanceInfo` 对象中读取数据：这个设置仅仅用于你想从 `metadata[cluster]` 中获取集群名，默认是 `appName` 也就是 `spring.application.name` 对应的值。  

  ```yaml
  # 如果你在eureka客户端中设置了metadata的cluster，则需要使用该配置项，否则没必要使用。
  eureka: 
    instance:
      metadata-map:
        cluster: ribbon
  ```

- `turbine.endpoints.clusters.enabled` 配置接收 `true` 和 `false`：用于是否禁用 `/clusters` 端点。

## Turbine 的使用

1. 加入POM依赖：  

   ```xml
   <dependency>
   	<groupId>org.springframework.cloud</groupId>
       <artifactId>spring-cloud-starter-netflix-turbine</artifactId>
   </dependency>
   ```

2. 在启动类上添加 `@EnableTurbine` 注解；

3. 添加 `Turbine` 的相关配置：  

   ```yaml
   turbine:
     app-config: system-management
     aggregator:
       cluster-config: SYSTEM-MANAGEMENT
     combine-host-port: true
   ```

4. 访问在Hystrix Dashboard输入 `http://hostname:port/turbine.stream?cluster=SYSTEM-MANAGEMENT` 查看监控（需要自己触发一下Hystrix的接口）。

5. Turbine提供了一个端点 `/clusters` ，用于查询所有已配置的集群信息，该端点返回一个JSON数组：  

   ```json
   [
     {
       "name": "RACES",
       "link": "http://localhost:8383/turbine.stream?cluster=RACES"
     },
     {
       "name": "WEB",
       "link": "http://localhost:8383/turbine.stream?cluster=WEB"
     }
   ]
   ```

