## 概述

到这一步，我已经租好了VPS，并且部署了博客，通过IP地址的方式可以通过互联网访问博客。  

但是IP不方便记忆，因此还得有好记又独一无二的域名，为了使域名能够绑定到IP，还需要DNS解析。  



## 域名

> 推荐阅读：[namesilo域名购买与使用](https://www.jianshu.com/p/27b0ebdcec2c)

### 选择

国内域名商主推阿里云吧，这里主要介绍国外的。国外域名即买即用无需备案，买了就是真正属于你的私有财产。但是如果你要使用国内的CDN加速和对象存储的话，那就在国内买域名吧，因为所有服务都要备案。  

国外主要的域名商有这几家：  

- GoDaddy：非常大的公司，第一年优惠特别大，续费的时候就呵呵了，不推荐。
- 1&1：看到它直接黑名单略过吧，无需考虑。
- porkbun：提供免费的SSL和WHOIS保护，稍微贵一点点。
- namesilo：【推荐】提供免费的WHOIS保护，可以使用1$的优惠卷，不会高价续费，反正听良心的。

这里推荐一个比价网站，[domcomp](https://www.domcomp.com/?refcode=5bf52f5b120000fd6a76e6ce)在这里可以看到所有域名商的价格比对，以及优惠信息，非常的方便。  

我自己是在namesilo购买的域名。



## DNS解析

DNS解析有两种方式：  

1. 在域名商那里解析：

    因为是国外的域名商，因此国内访问会稍慢，有时还会抽风。

2. 使用第三方的DNS服务商解析

    这里可以选择国内的DNS服务商解析，比如腾讯的[DNSDOP](https://www.dnspod.cn)，免费易用，还自带监控。  

    我用的国外的[cloudflare](https://www.cloudflare.com/)免费DNS解析，免费CDN加速，免费SSL。  

设置好DNS解析后，等一会就可以访问域名了。



## 修改Nginx配置

没有域名之前，一直配置的IP，现在将 `server_name` 指令的值改为自己的域名即可：   

```bash
server {
    ...
    server_name xxxx.com www.xxxx.com;
    ...
}
```

然后重新加载配置：  

```bash
sudo nginx -s reload
```



## 图床

现在七牛等服务商的对象存储，CDN加速都要备案才行了，对于图片的存放只有靠图床了，但是呢自己搭吧，怕麻烦，别人搭吧，怕跑路。emmm... 不想折腾的还是弄个国内的备案吧。  

- [cloudinary](https://cloudinary.com)：免费套餐 10g 空间，20g 流量，akamai 全球加速。
- [路过图床](https://imgchr.com/)：全球CND，支持外链,，无限空间，无限流量，原图保存，大佬维护。
- [SM图床](https://sm.ms/)：某大佬维护。
- [imgur](https://imgur.com/)：资源多，国内访问可能不太稳定。
- 新浪微博：嗯，你没看错，微博当作图床还是很稳定的。

我最后选择了路过图床，因为该大佬是真的有信仰，感谢他们的付出，让互联网变得更美好。  

  

## HTTPS

[cloudflare](https://www.cloudflare.com/)的免费SSL证书有效期长大15年，操作起来特别简单，按照官网的步骤一点一点来就好，图文教程见[这里](https://www.flyzy2005.com/build-page/cloudflare-free-https/)。  

大概的流程就是通过cloudflare生成pem证书和私匙，然后把证书和私匙放到服务器上去，修改Nginx配置，在你网站的虚拟服务 `server` 块中直接加入以下关键设置：  

```Nginx config files
# SSL
listen     443;
ssl        on;
ssl_certificate <pem文件路径>;
ssl_certificate_key <key私匙路径>;
```

最后在cloudflare上将加密设置为 `FULL(strict)` ，重启Nginx，就可以使用HTTPS啦！  

