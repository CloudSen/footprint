# Zuul处理跨域问题

默认情况下，Zuul会路由所有的跨域请求 `Cross-Origin Resource Sharing(CORS)` 。如果你希望自定义，可以提供一个 `WebMvcConfigurer` bean：  

```java
@Bean
public WebMvcConfigurer corsConfigurer() {
    return new WebMvcConfigurer() {
        public void addCorsMappings(CorsRegistry registry) {
            registry.addMapping("/path-1/**")
                    .allowedOrigins("https://allowed-origin.com")
                    .allowedMethods("GET", "POST");
        }
    };
}
```

在上面的例子中，允许 `https://allowed-origin.com/` 以GET和POST的方式发起 `/path-1/` 开头的跨域请求。你可以在这里设置 `allowedOrigins` 、`allowedMethods`、`allowedHeaders`、`exposedHeaders`、`allowCredentials` 和 `maxAge` 。