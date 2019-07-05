# Hystrix Dashboard

Hystrix会收集每个`@HystrixCommand` 的指标集。Hystrix Dashboard以高效的方式显示每个断路器的运行状况。  

## Hystrix Dashborad的使用

1. 加入POM依赖：  

   ```xml
   <dependency>
       <groupId>org.springframework.cloud</groupId>
       <artifactId>spring-cloud-starter-netflix-hystrix-dashboard</artifactId>
   </dependency>
   ```

2. 在启动类上加入 `@EnableHystrixDashboard` 注解；

3. 访问 `/hystrix` ，然后指定你要监控的 `/hystrix.stream` 应用。