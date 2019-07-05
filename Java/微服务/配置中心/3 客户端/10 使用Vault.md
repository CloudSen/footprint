# 使用Vault

当你使用 `Vault` 作为云配置服务的后端时，云配置服务客户端需要提供一个 `token` 用于从服务端获取数据。  

这个token可以通过 `bootstrap.yml` 配置文件中的 `spring.cloud.config.token` 来配置：  

***bootstrap.yml***  

```yaml
spring:
  cloud:
    config:
      token: YourVaultToken
```

## 在Vault中嵌入密钥

Vault提供了在其存储的之中嵌入密钥的支持：  

```
echo -n '{"appA": {"secret": "appAsecret"}, "bar": "baz"}' | vault write secret/myapp -
```

上面的指令将一个JSON对象写入了Vault，你可以在Spring应用中通过 `.` 来访问：  

```java
@Value("${appA.secret}")
String name = "World";
```

以上代码会将 `name` 变量的值设置为 `appAsecret` 。