# Ribbon Load Balancer 负载均衡

Spring Cloud Ribbon是基于Netflix Ribbon实现的一套***基于HTTP和TCP的客户端负载均衡工具***。它不像服务注册中心、 配置中心、 API 网关那样需要独立部署，它存在于每个微服务客户端中。另一个负载均衡工具`Feign` 也使用了 `Ribbon` ，因此如果你在使用 `@FeignClient` 这篇文章也是有用的。   

它提供了***客户端的软件负载均衡算法*** ，也可以很容易的加入自定义的负载均衡算法，拥有一系列完善的配置项如连接超时，重试等。  

> 负载均衡的算法规则一般有：简单轮询、随即连接、根据相应时间加权等。  
>
> Ribbon默认使用轮询算法。  

`Ribbon` 的核心概念是 `named client` 。每个负载均衡器都是某个组件集合的一部分，它的作用是根据需求与远程服务建立连接，作为开发人员，你会给这个组件集合一个名字，例如使用 `@FeignClient` 注解。Spring Cloud通过 `RibbonClientConfiguration` 为每个 `named client` 创建 `ApplicationContext` 。主要包含了 `ILoadBalancer` 、`RestClient` 和 `ServerListFilter`。  



## 如何在应用中加入Ribbon

在POM添加对应的依赖：  

```xml
<dependency>
	<groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-ribbon</artifactId>
</dependency>
```



## 自定义Ribbon客户端

你可以通过 `<client>.ribbon.*` 来对Ribbon客户端进行部分控制，就像使用原生Netflix APIs一样。这些原生配置可以视为 `CommonClientConfigKey` 的静态字段。  

Spring Cloud还允许你通过 `@RibbonClient` 注解申明额外的配置类（基于 `RibbonClientConfiguration`），来获得Ribbon客户端的完全控制：  

```java
@Configuration
@RibbonClient(name = "custom", configuration = CustomConfiguration.class)
public class TestConfiguration {
    // ...
}
```

上面的例子中，`@RibbonClient` 申明的客户端由 `RibbonClientConfiguration` 的组件组成，然后加入了额外的 `CustomConfiguration` 组件（后者会覆盖前者的配置）。  

> 【注意】：  
>
> 1. `CustomConfiguration` 类必须是一个 `@Configuration` 配置类；
> 2. `CustomConfiguration` 不能被主应用上下文的 `@ComponentScan` 所扫描到；
>
> 否则这个 `CustomConfiguration` 会被所有的 `@RibbonClients` 共享。如果使用Spring Boot，那么最好的解决方法是新建一个与主启动类同级的包，然后在这个包里添加自己的Ribbon自定义配置类。

下面的表格展示了Spring Cloud Netflix 提供给Ribbon的组件：  

|          Bean类型          |         Bean名字          |               类名               |
| :------------------------: | :-----------------------: | :------------------------------: |
|      `IClientConfig`       |   `ribbonClientConfig`    |    `DefaultClientConfigImpl`     |
|          `IRule`           |       `ribbonRule`        |       `ZoneAvoidanceRule`        |
|          `IPing`           |       `ribbonPing`        |           `DummyPing`            |
|    `ServerList<Server>`    |    `ribbonServerList`     |  `ConfigurationBasedServerList`  |
| `ServerListFilter<Server>` | `ribbonServerListFilter`  | `ZonePreferenceServerListFilter` |
|      `ILoadBalancer`       |   `ribbonLoadBalancer`    |     `ZoneAwareLoadBalancer`      |
|    `ServerListUpdater`     | `ribbonServerListUpdater` |    `PollingServerListUpdater`    |

创建一个自定义的 `CustomConfiguration` 类，然后在这个类中创建以上表格的Bean，最后将 `CustomConfiguration` 传给 `@RibbonClient` 注解的 `configuraion` 属性中，就可以重写对应的组件。  

下面的例子重写了两个默认组件：    

```java
@Configuration
protected static class CustomConfiguration {
    @Bean
    public ZonePreferenceServerListFilter serverListFilter() {
        ZonePreferenceServerListFilter filter = new ZonePreferenceServerListFilter();
        filter.setZone("myTestZone");
        return filter;
    }
    @Bean
    publlic IPing ribbonPing() {
        return new PingUrl();
    }
}
```



## 给所有的Ribbon客户端自定义默认配置

可以通过 `@RibbonClients` 注解来为所有Ribbon客户端定义一个默认的配置。如下所示：  

```java
@RibbonClients(defaultConfiguration = DefaultRibbonConfig.class)
public class RibbonClientDefaultConfigurationTestsConfig {

	public static class BazServiceList extends ConfigurationBasedServerList {

		public BazServiceList(IClientConfig config) {
			super.initWithNiwsConfig(config);
		}

	}

}

@Configuration
class DefaultRibbonConfig {

	@Bean
	public IRule ribbonRule() {
		return new BestAvailableRule();
	}

	@Bean
	public IPing ribbonPing() {
		return new PingUrl();
	}

	@Bean
	public ServerList<Server> ribbonServerList(IClientConfig config) {
		return new RibbonClientDefaultConfigurationTestsConfig
            .BazServiceList(config);
	}

	@Bean
	public ServerListSubsetFilter serverListFilter() {
		ServerListSubsetFilter filter = new ServerListSubsetFilter();
		return filter;
	}

}
```



## 通过配置属性来自定义Ribbon客户端

通过配置属性，允许你针对不同的环境， 在启动时改变Ribbon行为。下面是支持的属性：  

- `<clientName>.ribbon.NFLoadBalancerClassName` ：需要 `ILoadBalancer` 的实现类；
- `<clientName>.ribbon.NFLoadBalancerRuleClassName` ：需要 `IRule` 的实现类；
- `<clientName>.ribbon.NFLoadBalancerPingClassName` ：需要 `IPing` 的实现类；
- `<clientName>.ribbon.NIWSServerListClassName` ：需要 `ServerList` 的实现类；
- `<clientName>.ribbon.NIWSServerListFilterClassName` ：需要 `ServerListFilter` 的实现类；

> 属性配置的优先级比注解 `@RibbonClient(configuration = xxx.class)` 更高。

例如：为名为 `users` 的Ribbon客户端指定 `IRule` ，可以像下面这样设置：  

```yaml
users:
  ribbon:
    NIWSServerListClassName: com.netflix.loadbalancer.ConfigurationBasedServerList
    NFLoadBalancerRuleClassName: com.netflix.loadbalancer.WeightedResponseTimeRule
```



## IRule 负载均衡算法接口

默认提供了以下几种LB算法实现类：  

- RoundRobinRule：默认规则，轮询；

- RandomRule：随机；

- AvailabilityFilteringRule：

- WeightedResponseTimeRule：

  根据平均相应时间加权重，相应时间越快，权重越高；  

  刚启动时若统计信息不足，则使用RoundRobinRule，直到信息充足；

- RetryRule：

  先按照RoundRobinRule获取服务，若获取失败，则在指定时间内会进行重试；

- BestAvailableRule：

- ZoneAvoidanceRule：复合判断服务所在区域的性能和可用性选择服务器；

