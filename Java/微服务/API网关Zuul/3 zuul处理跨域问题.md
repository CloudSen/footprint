# Zuul处理跨域问题

浏览器为了网络完全，避免被恶意攻击，默认是不允许跨域访问的。比如在 `http:localhost:8700` 这台服务器上运行的web应用，访问 `http://localhost:8222` 服务器上的应用时，前端会返回这样的错误信息：  

```
Access to XMLHttpRequest at 'http://localhost:8222/api/demo-for-test' from origin 'http://localhost:8700' has been blocked by CORS policy: No 'Access-Control-Allow-Origin' header is present on the requested resource.
```



## zuul官方文档的说明

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



## Spring框架的说明

你可以通过Spring内置的 `CorsFilter` 来处理 `CORS` 。  

> 当你在含有 `Spring Security` 的项目中使用 `CorsFilter` 的时候，请注意，Spring Security已经有 `CORS` 内建支持了。见[这里](https://docs.spring.io/spring-security/site/docs/current/reference/htmlsingle/#cors)。

配置 `CorsFilter` ，只需给它的构造函数传入一个 `CorsConfigurationSource` ：  

```java
CorsConfiguration config = new CorsConfiguration();
// Possibly...
// config.applyPermitDefaultValues()
config.setAllowCredentials(true);
config.addAllowedOrigin("http://domain1.com");
config.addAllowedHeader("");
config.addAllowedMethod("");
UrlBasedCorsConfigurationSource source = new UrlBasedCorsConfigurationSource();
source.registerCorsConfiguration("/**", config);
CorsFilter filter = new CorsFilter(source);
```



## Spring Security对CORS的内建支持

> [Spring-Security-CORS](https://docs.spring.io/spring-security/site/docs/current/reference/htmlsingle/#cors)

在security配置类中可以这样配置：  

```java
@EnableWebSecurity
public class WebSecurityConfig extends WebSecurityConfigurerAdapter {

    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http
            // by default uses a Bean by the name of corsConfigurationSource
            .cors().and()
            ...
    }

    // 写一个CORS配置源bean
    @Bean
    CorsConfigurationSource corsConfigurationSource() {
        CorsConfiguration configuration = new CorsConfiguration();
        configuration.setAllowedOrigins(Arrays.asList("https://example.com"));
        configuration.setAllowedMethods(Arrays.asList("GET","POST"));
        UrlBasedCorsConfigurationSource source = new UrlBasedCorsConfigurationSource();
        source.registerCorsConfiguration("/**", configuration);
        return source;
    }
}
```

