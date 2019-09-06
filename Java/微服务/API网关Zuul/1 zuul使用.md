# Zuul的使用

1. 在POM依赖中加入：  

   ```xml
   <dependency>
       <groupId>org.springframework.cloud</groupId>
       <artifactId>spring-cloud-starter-netflix-zuul</artifactId>
   </dependency>
   <dependency>
       <groupId>org.springframework.cloud</groupId>
       <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
   </dependency>
   ```

2. 在启动类上添加 `EnableZuulProxy` 注解，代表这个应用是Zuul服务端：  

   ```java
   /**
    * API路由
    *
    * @author CloudSen
    */
   @Slf4j
   @EnableEurekaClient
   @EnableZuulServer
   @SpringBootApplication
   public class ZuulApplication {
       static {
           log.info("======> [ INFO ] Starting demo server...");
       }
   
       public static void main(String[] args) {
           new SpringApplicationBuilder(ZuulApplication.class)
                   .web(WebApplicationType.SERVLET)
                   .run(args);
       }
   }
   
   ```

   > `@EnableZuulProxy` 和 `@EnableZuulServer` 两个注解的区别是：  
   >
   > EnableZuulProxy 注解比后者多了以下过滤器：  
   >
   > - pre类型过滤器：PreDecorationFilter
   > - route类型过滤器：RibbonRoutingFilter，SimpleHostRoutingFilter
   >
   > `@EnableZuulProxy` 简单理解为 `@EnableZuulServer` 的增强版，当Zuul与Eureka、Ribbon等组件配合使用时，我们使用 `@EnableZuulProxy`。

3. 编写配置文件，加入路由相关配置：  
    ```yaml
    zuul:
      prefix: /api
      retryable: true
      ignoredServices: '*'
      ignoredPatterns: /**/admin/**
      routes:
        cloudable-blog: /cloudable-blog/**
    management:
      endpoints:
        web:
          exposure:
            include: health, info, routes, filters
    ```


