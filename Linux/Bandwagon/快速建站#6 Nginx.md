> 之前的文章已经Nginx Open Source做了详细介绍，请自行查阅。

## 安装

```bash
sudo apt-get update
sudo apt-get install nginx
sudo nginx -v
```

## 创建配置文件

```bash
mkdir -p ~/work/deploy/webservers/nginx/
vim ~/work/deploy/webservers/nginx/blog_nginx.conf
```

blog_nginx.conf：  

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
	sendfile off;
	# 请求访问的日志路径
    access_log /home/cloudsen/work/deploy/webservers/nginx/access.log combined;
    # Arch Linux 需要的额外设置 start
    types_hash_max_size 4096;
    server_names_hash_bucket_size 128;
    # Arch Linux 需要的额外设置 end
    
    upstream app_server {
        # unix域的套接字配置
        # fail_timeout=0作用是即使失败也要不断尝试上游服务器返回一个好的HTTP相应
        server unix:/home/cloudsen/work/deploy/webservers/sockets/nginx-gunicorn.sock fail_timeout=0;
        # TCP的配置如下
    	# server 192.168.0.7:8000 fail_timeout=0;
    }
    
    server {
        # 如果没有HOST匹配，则关闭链接防止HOST欺骗
        listen 80 default_server;
        return 444;
    }
    
    server {
        listen 80;
        server_name <你的服务器host>;
        client_max_body_size 4G;
        keepalive_timeout 5;
        
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
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header X-Forwarded-Proto $scheme;
            proxy_set_header Host $http_host;
            proxy_pass_header Authorization;
        	proxy_pass_header WWW-Authenticate;
            proxy_redirect off;
            proxy_pass http://app_server;
        }
    }
}
```

## 测试配置文件正确性

```bash
sudo nginx -t -c ~/work/deploy/webservers/nginx/blog_nginx.conf

nginx: the configuration file /home/cloudsen/work/deploy/webservers/nginx/blog_nginx.conf syntax is ok
nginx: configuration file /home/cloudsen/work/deploy/webservers/nginx/blog_nginx.conf test is successful
```

## 启动Nginx

```bash
sudo nginx -c ~/work/deploy/webservers/nginx/blog_nginx.conf
```

如果80端口被占用，查看一下谁占用的：  

```bash
sudo netstat -tulpn | grep --color :80
```

如果修改了配置文件，只需要reload即可：  

```bash
sudo nginx -s reload
```

## 访问网站

目前还未购买域名，但可以通过IP直接访问，测试Nginx配置是否正确。