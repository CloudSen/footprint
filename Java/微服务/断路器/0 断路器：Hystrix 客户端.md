# 断路器：Hystrix 客户端

Netflix已经创建了一个名为 `Hystrix` 的库，它实现了马丁弗勒提到的断路器这部分内容。在一个微服务架构体系中，多层服务调用是非常常见的。  

在比较底层的服务一旦崩溃，它能沿着调用链，导致一系列的级联崩溃。为了解决这个问题，断路器就会监听崩溃，当到达一定条件后，直接跳闸，中断崩溃服务的访问，默认条件如下：  

- 超过 `circuitBreaker.requestVolumeThreshold` 的值，默认是20次request请求；
- 在 `metrics.rollingStats.timeInMilliseconds` 定义的一段时间内（窗口期），默认是10秒，失败率大于 `circuitBreaker.errorThresholdPercentage` ，默认是50%；

***断路器跳闸，可以阻止级联故障，允许崩溃的服务有时间去恢复故障。***  

注意，在发生错误和跳闸的情况下，开发人员可以提供一个 `Fallback` ，`Fallback` 可以是一个被 `Hystrix` 保护的调用，静态数据或一个有意义的空值。  



## 如何使用 Hystrix

1. 引入POM依赖：  

   ```xml
   <dependency>
       <groupId>org.springframework.cloud</groupId>
       <artifactId>spring-cloud-starter-netflix-hystrix</artifactId>
   </dependency>
   ```

2. 在启动类上添加启用断路器的注解 `@EnableCircuitBreaker` ：  

   ```java
   @Slf4j
   @EnableEurekaClient
   @EnableCircuitBreaker
   @EnableSpringDataWebSupport
   @SpringBootApplication
   public class SysManagementApplication {
       static {
           log.info("======> [ INFO ] Starting system management server...");
       }
   
       public static void main(String[] args) {
           new SpringApplicationBuilder(SysManagementApplication.class)
                   .web(WebApplicationType.SERVLET)
                   .run(args);
       }
   }
   ```

3. 在需要断路器的地方，通过 `@HystrixCommand` 注解指定 `fallbackMethod` ：  

   ```java
   @Component
   public class StoreIntegration {
   
       @HystrixCommand(fallbackMethod = "defaultStores")
       public Object getStores(Map<String, Object> parameters) {
           //do stuff that might fail
       }
   
       public Object defaultStores(Map<String, Object> parameters) {
           return /* something useful */;
       }
   }
   ```

`@HystrixCommand` 由Netflix的一个名叫 `javanica` 的贡献库提供。Spring Cloud 会自动将拥有该注解的Bean，包装到连接 `Hystrix` 断路器的代理对象中。在发生故障的时候，断路器会通过计算来决定什么时候开启和关闭跳闸，并决定如何处理当前故障。  

`@HystrixCommand` 注解中，可以使用带有 `@HystrixProperty` 注解列表的 `commandProperties` 属性：  

```java
@HystrixCommand(fallbackMethod = "stubMyService",
    commandProperties = {
      @HystrixProperty(name="execution.isolation.strategy", value="SEMAPHORE")
    }
)
```



## 健康检测

已连接的断路器状态，暴露在调用应用程序的 `/health` 端点中，如以下示例所示：  

```json
{
    "hystrix": {
        "openCircuitBreakers": [
            "StoreIntegration::getStoresByLocationLink"
        ],
        "status": "CIRCUIT_OPEN"
    },
    "status": "UP"
}
```



## Hystrix metrics 指标流

启用 `Hystrix metrics stream` ：  

1. 加入POM依赖：  

   ```xml
   <dependency>
       <groupId>org.springframework.boot</groupId>
       <artifactId>spring-boot-starter-actuator</artifactId>
   </dependency>
   ```

2. 加入 `actuator` 配置：

   这会将 `/actuator/hystrix.stream` 访问路径暴露在外。  

   ```properties
   management.endpoints.web.exposure.include: hystrix.stream
   ```

3. 访问 `/actuator/hystrix.stream`：  

   没有断路发生时，一直处于 `ping` 状态；短路发生后，能看到一大片数据，此时需要使用 `Hystrix Dashboard` 来监控。  

## Bibbon和Hystrix的超时

当你使用 `HystrixCommand` 包装 `Ribbon` 时，你需要确保 `Hystrix` 的超时时间大于 `Ribbon` 的超时时间（包含了潜在的重试次数）。比如，你的 `Ribbon` 超时时间设置的1秒，然后设置了重试次数是3次，那么你的 `Hystrix` 超时时间一定是要大于3秒。  

 

