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

2. 在启动类上添加 `@EnableZuulServer` 注解，代表这个应用是Zuul服务端：  

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

   

