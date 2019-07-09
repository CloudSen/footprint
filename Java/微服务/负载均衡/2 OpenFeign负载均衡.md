# OpenFeign负载均衡

`Feign` 是一个声明式的web客户端。它使编写web服务客户端变得更加容易。要使用 Feign ，你需要创建一个接口并且给它添加注解。它支持可插拔的注解配置，包括Feign注解和 `JAX-RS` 注解（Java API for RESTful Web Services）。Feign还支持可插拔的编码解码器。当使用Feign时，可以使用MVC模块的相关注解，Spring Cloud集成了Ribbon和Eureka，并提供基于http的负载均衡客户端。  

## 如何使用Feign

1. 在POM中加入依赖：  

```xml
<dependency>
	<groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-openfeign</artifactId>
</dependency>
```

2. 在启动类上添加 `@EnableFeignClients` 注解；

3. 新建接口并通过 `@FeignClient` 等注解，定义Feign：  

   ```java
   @FeignClient("stores")
   public interface StoreClient {
       @RequestMapping(method = RequestMethod.GET, value = "/stores")
       List<Store> getStores();
   
       @RequestMapping(method = RequestMethod.POST, value = "/stores/{storeId}", consumes = "application/json")
       Store update(@PathVariable("storeId") Long storeId, Store store);
   }
   ```

   `@FeignClient` 注解接收一个字符串，它表示了客户端名字（跟 `spring.application.name对应` ），用于创建一个Ribbon负载均衡器。你也可以使用 `url` 参数指定一个URL地址（可以是完整地址或者hostname）。在 `application context` 中的bean的名字就是这个接口名。若要自定义bean的名字，可以通过 `qualifier` 属性指定。  

   上面创建的Ribbon负载均衡器会去发现名为stores服务的物理地址。如果你使用了Eureka，它会在Eureka注册中心获取。如果没有使用Eureka，你可以通过简单的外部配置来指定一个服务列表（[详情见Ribbon配置说明](https://cloud.spring.io/spring-cloud-static/Greenwich.SR2/multi/multi_spring-cloud-ribbon.html#spring-cloud-ribbon-without-eureka)）。  



## 重写Feign默认配置

`Feign` 的概念和 `Ribbon` 一样，核心都是 `named client` 。每个Feign客户端都是一个组件集合的一部分，它的作用是根据需求与远程服务建立连接。开发人员通过 `@FeignClient` 注解给组件集合定义一个名字。Spring Cloud通过 `FeignClientsConfiguration` 为每个 `named client` 创建 `ApplicationContext` 。它主要包含 `feign.Decoder` 、`feign.Encoder` 和 `feign.Contract` 。你可以通过 `@FeignClient` 注解的 `contextId` 来定义这个组件集合的名字。  

Spring Cloud允许你通过 `@FeignClient` 注解申明一个自定义配置（基于 `FeignClientsConfiguration`），来获得Feign客户端的完全控制：  

```java
@FeignClient(name = "stores", configuration = FooConfiguration.class)
public interface StoreClient {
    // ...
}
```

上面的例子中，`@FeignClient` 申明的客户端由 `RibbonClientConfiguration` 的组件组成，然后加入了额外的 `FooConfiguration` 组件（后者会覆盖前者的配置）。  

> 【注意】：  
>
> 1. `FooConfiguration` 不需要加上 `@Configuration` 注解；
> 2. 如果 `FooConfiguration` 被加上了 `@Configuration` 注解，则需要排除在 `ComponentScan` 之外；

> `serviceId` 属性已经被弃用，推荐使用 `name` 属性。

> 以前，使用 `url` 属性时，不需要 `name` 。现在，必须拥有 `name` 。

> `contextId` 和 `name` 的区别是：`contextId` 不但设置了组件集合在 `application context` 中的名字，还重写了客户端名称的别名，并将用作该客户端创建的bean名称的一部分。

`name` 和 `url` 支持占位符：  

```java
@FeignClient(name = "${feign.name}", url = "${feign.url}")
public interface StoreClient {
    // ...
}
```

Spring Cloud Netflix为Feign默认提供了下面的组件：  

|    Bean类型     |    Bean名     |                             类名                             |
| :-------------: | :-----------: | :----------------------------------------------------------: |
|    `Decoder`    | feignDecoder  |       `ResponseEntityDecoder` （包装了SpringDecoder）        |
|    `Encoder`    | feignEncoder  |                       `SpringEncoder`                        |
|    `Logger`     |  feignLogger  |                        `Slf4jLogger`                         |
|   `Contract`    | feignContract |                     `SpringMvcContract`                      |
| `Feign.Builder` | feignBuilder  |                    `HystrixFeign.Builder`                    |
|    `Client`     |  feignClient  | 启用Ribbon时是 `LoadBalancerFeignClient` ；否则是默认的feign客户端 |

Spring Cloud Netflix没有提供以下的组件，但是你可以自己提供：  

- `Logger.Level`
- `Retryer`
- `ErrorDecoder`
- `Request.Options`
- `Collection<RequestInterceptor>`
- `SetterFactory`

想要创建上面的组件，可以在自己的feign配置类中重写，例如：  

```java
@Configuration
public class FooConfiguration {
    @Bean
    public Contract feignContract() {
        return new feign.Contract.Default();
    }

    @Bean
    public BasicAuthRequestInterceptor basicAuthRequestInterceptor() {
        return new BasicAuthRequestInterceptor("user", "password");
    }
}
```

你也可以通过配置文件来配置Feign：  

***application.yml***  

```yaml
feign:
  client:
    config:
      feignName:
        connectTimeout: 5000
        readTimeout: 5000
        loggerLevel: full
        errorDecoder: com.example.SimpleErrorDecoder
        retryer: com.example.SimpleRetryer
        requestInterceptors:
          - com.example.FooRequestInterceptor
          - com.example.BarRequestInterceptor
        decode404: false
        encoder: com.example.SimpleEncoder
        decoder: com.example.SimpleDecoder
        contract: com.example.SimpleContract
```

若上面的 `feignName` 改为 `default` ，则它会用于所有的 `@FeignClient` 配置。  

***配置文件的优先级默认是高于配置类的***，可以通过 `feign.client.default-to-properties=false` 来改变。  

