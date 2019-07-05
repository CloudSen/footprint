# 提供自定义RestTemplate

在一些场景下，你也许需要自定义云配置客户端向服务端发送的请求，通常是为了传递 `Authorization` 授权头，那么你需要自定义 `RestTemplate`。  

创建新的配置Bean，并实现  `PropertySourceLocator` ：  

***CustomConfigServiceBootstrapConfiguration.java***  

```java
@Configuration
public class CustomConfigServiceBootstrapConfiguration {
    @Bean
    public ConfigServicePropertySourceLocator configServicePropertySourceLocator() {
        ConfigClientProperties clientProperties = configClientProperties();
        ConfigServicePropertySourceLocator configServicePropertySourceLocator = 
            new ConfigServicePropertySourceLocator(clientProperties);
        configServicePropertySourceLocator
            .setRestTemplate(customRestTemplate(clientProperties));
        return configServicePropertySourceLocator;
    }
}
```

在 `resources/META-INF` 路径下，创建名为 `spring.factories` 的文件，然后指明自定义配置：  

***spring.factories***  

```
org.springframework.cloud.bootstrap.BootstrapConfiguration = com.my.config.client.CustomConfigServiceBootstrapConfiguration
```

