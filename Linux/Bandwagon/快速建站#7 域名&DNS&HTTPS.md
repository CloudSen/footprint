# 嘿，等等

回想一下，到这一步，我已经租好了VPS，并且使用Gunicorn配置Nginx部署了博客，能够通过IP地址和URI的方式，使用互联网访问博客。  

但是IP不方便记忆，因此，还得给IP取个别名，这个别名就是域名，为了使域名能够绑定到IP，还需要DNS解析。  



# 域名服务商的选择

> 推荐阅读：[namesilo域名购买与使用](https://www.jianshu.com/p/27b0ebdcec2c)

国内的域名商主推阿里云吧，这里主要介绍国外的。国外域名即买即用，无需备案，是真正属于你的私有财产。但是，请注意，如果你要使用国内的CDN加速和对象存储的话，那就在国内买域名吧，因为所有服务都要备案。  

国外的域名商主要有这几家：  

- GoDaddy(狗爹)：非常大的公司，第一年优惠特别大，续费的时候就呵呵了，够狗，不推荐。
- 1&1：看到它直接黑名单略过吧，无需考虑。
- porkbun：提供免费的SSL和WHOIS保护，稍微贵一点点。
- namesilo：【推荐】提供免费的WHOIS保护，可以使用1$的优惠卷，不会高价续费，反正挺良心的。

这里推荐一个域名比价网站，[domcomp](https://www.domcomp.com/?refcode=5bf52f5b120000fd6a76e6ce) ，在这里可以看到所有域名商的价格比对，以及优惠信息，非常的方便。  

我自己是在namesilo购买的域名。



# DNS解析

DNS解析有两种渠道：  

1. 在域名商那里直接解析：

    因为是国外的域名商，因此国内访问会稍慢，有时还会抽风。

2. 使用第三方的DNS服务商解析

    这里可以选择国内的DNS服务商解析，比如腾讯的[DNSDOP](https://www.dnspod.cn)，免费易用，还自带监控。  

    我用的国外的[cloudflare](https://www.cloudflare.com/)免费DNS解析，免费CDN加速，免费SSL，我也推荐各位使用它，缺点就是国内加速效果一般，但还是有效果的，毕竟是免费的，已经很好了。  

设置好DNS解析后，等一会儿你就可以直接输入域名，来访问服务器了。  

这里推荐一个在线测速工具：[IPIP](https://www.ipip.net/ip.html)  。



# 修改Nginx配置

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



# 图床

现在，七牛等服务商的对象存储，CDN加速都要备案才可以使用。那么图片的存放只有靠图床了，但是呢自己搭吧，怕麻烦，别人搭吧，怕跑路。emmm... 不想折腾的还是弄个国内的备案吧。  

- [cloudinary](https://cloudinary.com)：免费套餐 10g 空间，20g 流量，akamai 全球加速。
- [路过图床](https://imgchr.com/)：全球CND，支持外链,，无限空间，无限流量，原图保存，大佬维护。
- [SM图床](https://sm.ms/)：某大佬维护。
- [imgur](https://imgur.com/)：资源多，国内访问可能不太稳定。
- 新浪微博：嗯，你没看错，微博当作图床还是很稳定的。

我最后选择了路过图床，速度也很快。另外，该大佬是真的有信仰，感谢他们的付出，让互联网变得更美好。  

  

# HTTPS

[cloudflare](https://www.cloudflare.com/)免费 `SSL证书` 的有效期长达15年，操作起来特别简单，按照官网的步骤一点一点来就好，图文教程见[这里](https://www.flyzy2005.com/build-page/cloudflare-free-https/)。  

大概的流程就是通过cloudflare生成 `pem证书` 和 `私匙` ，然后把证书和私匙放到服务器上去，修改Nginx配置，在虚拟服务 `server` 块中直接加入以下关键设置：  

```Nginx config files
# SSL
listen     443;
ssl        on;
ssl_certificate <pem文件路径>;
ssl_certificate_key <key私匙路径>;
```

最后在cloudflare上将加密设置为 `FULL(strict)` ，重启Nginx，就可以使用HTTPS啦！  

附上我的最终Nginx配置：  

```Nginx config files
# 设置worker进程数
worker_processes 2;
# 指定用户
user cloudsen;
# 输出错误日志的路径 log过滤为警告
error_log /home/cloudsen/work/deploy/webservers/nginx/error.log warn;
# nginx运行时的PID
pid /var/run/nginx.pid;

events {
    # 如果你有很多客户端，这里就多弄一些
    worker_connections 1024;
    # accept_mutex设置为'on'时，如果nginx worker_processes > 1
    # 'use epoll;' to enable for Linux 2.6+
    # 'use kqueue;' to enable for FreeBSD, OSX
    accept_mutex on;
    use epoll;
}

http {
    include /etc/nginx/mime.types;
    default_type application/octet-stream;
        sendfile on;
        # 请求访问的日志路径
    access_log /home/cloudsen/work/deploy/webservers/nginx/access.log combined;
    # Arch Linux 需要的额外设置 start
    types_hash_max_size 4096;
    server_names_hash_bucket_size 128;
    # Arch Linux 需要的额外设置 end
    
    upstream app_server {
        # unix域的套接字配置
        # fail_timeout=0作用是即使失败也要不断尝试上游服务器返回一个好的HTTP相应
        #server unix:/home/cloudsen/work/deploy/webservers/sockets/nginx-gunicorn.sock fail_timeout=0;
        # TCP的配置如下
        server 127.0.0.1:8000 fail_timeout=0;
    }
    
    server {
        # 如果没有HOST匹配，则关闭链接防止HOST欺骗
        listen 80 default_server;
        return 444;
    }
    
    # http自动转到https
    server {
        listen 80;
        server_name *.yangyunsen.com;
        return 301 https://www.yangyunsen.com;
    }
    
    server {
        listen 443;
        server_name *.yangyunsen.com;
        client_max_body_size 4G;
        keepalive_timeout 5;

        # force SSL 强制使用HTTPS
        ssl on;
        ssl_certificate /home/cloudsen/work/ssl/yangyunsen.com.pem;
        ssl_certificate_key /home/cloudsen/work/ssl/yangyunsen.com.key;
        ssl_session_cache shared:SSL:10m;
        error_page 497 https://www.yanagyunsen.com;

        # 沒找到网站图标不报错
        location /favicon.ico {
            alias /home/cloudsen/work/deploy/webapps/RedQueen/collected_statics/;
                access_log off;
                log_not_found off;
        }
        
        # Nginx处理静态文件
        location /static/ {
            # 使用root最后是在/home/cloudsen/work/deploy/webapps/RedQueen/static中获取静态资源
            # root /home/cloudsen/work/deploy/webapps/RedQueen/;
            # 使用alias最后是在/home/cloudsen/work/deploy/webapps/RedQueen/collected_statics/中获取静态资源
            # 我的Django项目执行python manage.py collectstatic指令时，目标文件夹是collected_statics
            alias /home/cloudsen/work/deploy/webapps/RedQueen/collected_statics/;
        }
        
        # 精确匹配没有uri的连接，重写为我的主页uri
        location = / {
            rewrite ^ /cloudsen_blog;
        }
        
        # 其他的url代理给Gunicorn使用的 unix domain socket
        location / {
            proxy_pass_header X-CSRFToken;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header X-Forwarded-Proto $scheme;
            proxy_set_header Host $http_host;
            proxy_set_header Cookie $http_cookie;
            proxy_redirect off;
            proxy_pass http://app_server;
        }
    }
}

```

像我这样配置后，`http://www.yangyunsen.com` 自动变为 `htps://...`。`https://yangyunsen.com` 和 `http://yangyunsen.com` 直接拒绝连接。



# CDN加速

因为我是国外域名，没有备案，因此无法使用国内的CDN。这里依然使用cloudflar来实现，图文教程见这里[cloudflare免费CDN加速](https://www.jianshu.com/p/95a8f8e28649)。  

这是我加速前的情况，180ms~340ms多延迟：  

[![FiZ89U.md.png](https://s1.ax1x.com/2018/11/23/FiZ89U.md.png)](https://imgchr.com/i/FiZ89U)  

这是我加速后的情况，平均200ms以下了：  

[![FiZG3F.md.png](https://s1.ax1x.com/2018/11/23/FiZG3F.md.png)](https://imgchr.com/i/FiZG3F)  

嗯，看来还是有效果的。