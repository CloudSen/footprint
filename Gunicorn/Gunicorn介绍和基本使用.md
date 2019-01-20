[TOC]

# Gunicorn介绍

Gunicorn 的名字来源于 `Green Unicorn绿色独角兽` ，它是一个适用于UNIX的Python WSGI HTTP服务器。Gunicorn与各种web框架兼容，部署简单、服务器资源更明确并且速度相当快。默认是同步工作，支持Gevent、Eventlet异步。**强烈建议在Nginx代理服务器后面使用Gunicorn**。

## 特点

- 原生支持 `WSGI` , `Django` 与 `Paster`
- 自动化worker进程管理
- 简单的Python配置
- 多worker配置
- 各种服务器钩子，扩展性极强
- 兼容Python 2.x >= 2.6 或者 3.x >= 3.2



# Installation安装

## 安装

通过pip安装(建议安装在虚拟环境中)：  

```bash
pip install gunicorn
```

通过系统安装：  

```bash
# Ununtu
sudo apt-get install gunicorn
# Arch Linux
sudo pacman -S gunicorn
# Debian
sudo apt-get -t stretch-backports install gunicorn
```

## 异步的worker

> 这里涉及的是Python的协程

Gunicorn默认使用的同步worker，在请求处理期间，其他的请求会等待，相应速度很慢。若希望某些应用代码在请求处理期间暂停一会，去处理其他的事情，则需要异步worker。  

Gunicorn可以使用 `Eventlet` 和 `Gevent` ，这里使用 `Gevent` 讲解。  

```bash
# eventlet和gevent都需要它
pip install greenlet
pip install gevent
```

> 如果 `greenlet` 安装失败，请检查系统是否安装得有 `python-dev` 。
>
> ArchLinux 不需要也不能安装 `python-dev` ，因为已经包含在python包中了，Ubuntu可能会需要安装。
>
> `Gevent` 还需要安装 `libevent` 1.4.x或者2.0.4以上。

  

# 运行Gunicorn

你可以通过命令行的方式运行Gunicorn，也可以嵌入Django或Paster中去使用。  

## 命令行方式

当 `gunicorn` 安装完毕后，就可以使用命令行了。若通过pip安装的，则在虚拟环境中运行；若通过系统安装的，则可以通过终端直接执行。  

### gunicorn

基础用法：  

```bash
gunicorn [OPTIONS] APP_MODULE
```

`APP_MODULE` 的格式为 `$(MODULE_NAME):$(VARIABLE_NAME)` 。其中，`MODULE_NAME` 可以是一个完整的点路径。`VARIABLE_NAME` 指在特定模块中的WSGI可调用对象。如下案例：  

```python
def app(environ, start_response):
    """简单应用对象"""
    data = b'Hello, World!\n'
    status = '200 OK'
    response_headers = [
        ('Content-type', 'text/plain'),
        ('Content-Length', str(len(data)))
    ]
    start_response(status, response_headers)
    return iter([data])
```

现在你可以通过下面的命令行运行该应用：  

```bash
gunicorn --workers=2 test:app

[2018-11-19 03:16:08 +0800] [20440] [INFO] Starting gunicorn 19.9.0
[2018-11-19 03:16:08 +0800] [20440] [INFO] Listening at: http://127.0.0.1:8000 (20440)
[2018-11-19 03:16:08 +0800] [20440] [INFO] Using worker: sync
[2018-11-19 03:16:08 +0800] [20443] [INFO] Booting worker with pid: 20443
[2018-11-19 03:16:08 +0800] [20444] [INFO] Booting worker with pid: 20444
```

访问 `http://127.0.0.1:8000` 就能看到"Hello, World!" 。那么说明gunicorn正常运行。

### 常用参数

- `-c CONFIG, --config=CONFIG` - 通过以下方式指定一个配置文件，`$(PATH)` 或 `file:$(PATH)` 或 `python:$(MODULE_NAME)` 。
- `-b BIND, --bind=BIND` - 指定要绑定的服务器嵌套字。服务器嵌套字可以用以下格式：`$(HOST)` , `$(HOST):$(POST)` 或者 `unix:$(PATH)` 。一个IP是一个有效的$(HOST)。
- `-w WORKERS, --workers=WORKERS` - 设置worker进程的数量。每个CPU核心能分2~4个worker。
- `-k WORKERCLASS, --worker-class=WORKERCLASS` - worker进程的类型。你能将它设置为 `$(NAME)` ，这个$(NAME)可以是 `sync` , `eventlet` , `gevent` , `tornado` , `gthread` , `gaiohttp(deprecated)` 。`sync` 是默认的，worker同步阻塞运行，效率低。 

- `-n APP_NAME, --name=APP_NAME` - 如果安装了 `setproctitle` python模块，则可以调整Gunicorn进程的名字。

## 集成方式

### Django

Gunicorn默认查找名为 `application` 的可供WSGI调用的对象。对于典型的Django项目，调用Gunicorn如下所示：  

```bash
gunicorn myproject.wsgi
```

你可以使用 `--env` 来指定Django的配置文件：  

```bash
gunicorn --env DJANGO_SETTINGS_MODULE=myproject.settings myproject.wsgi
```

## 实际操作

```bash
gunicorn -w 4 -k gevent  RedQueen.wsgi:application

[2018-11-19 04:14:13 +0800] [23546] [INFO] Starting gunicorn 19.9.0
[2018-11-19 04:14:13 +0800] [23546] [INFO] Listening at: http://127.0.0.1:8000 (23546)
[2018-11-19 04:14:13 +0800] [23546] [INFO] Using worker: gevent
[2018-11-19 04:14:13 +0800] [23550] [INFO] Booting worker with pid: 23550
[2018-11-19 04:14:13 +0800] [23551] [INFO] Booting worker with pid: 23551
[2018-11-19 04:14:13 +0800] [23552] [INFO] Booting worker with pid: 23552
[2018-11-19 04:14:13 +0800] [23553] [INFO] Booting worker with pid: 23553
```

访问http://127.0.0.1:8000后，发现无法加载静态文件。这里可以暂时使用 `Whitenoise` 来解决，以后会使用 `Nginx` 来帮忙处理静态文件。  



# 配置概述

Gunicorn从三个地方拉取配置信息，优先级从低到高为：   

- web框架的配置（目前只支持Paster应用框架）
- 配置文件
- 命令行

## 命令行

优先级最高，如果在命令行中指定了一个选项，它将覆盖在应用程序中或配置文件中所指定的所有其他设置。并非所有Gunicorn设置都可以从命令行设置。要查看命令行设置的完整列表，你可以执行以下操作：  

```bash
gunicorn -h
```

## 配置文件

配置文件应该是有效的**Python源文件**。它只需要在文件系统中可读即可。更具体地说，它不需要是可导入的。任何Python都是有效的。只要考虑每次启动Gunicorn时都会运行（包括当你指示Gunicorn重新加载时）。要设置参数，没有特殊的语法，只需用Python语法赋值即可。  

例如，在django项目中创建 `gunicorn_conf.py` ：  

```bash
import multiprocessing

# unix domain socket方式
# unix:千万不要写掉了。确保路径文件夹存在，sock文件会自动创建和销毁，不要手动创建。
bind = 'unix:/home/cloudsen/work/deploy/webservers/sockets/nginx-gunicorn.sock'
# tcp方式
#bind = '127.0.0.1:8000'
workers = multiprocessing.cpu_count() * 2 + 1
worker_class = 'gevent'
errorlog = '/home/cloudsen/work/deploy/webservers/gunicorn/guicorn.error.log'
accesslog = '/home/cloudsen/work/deploy/webservers/gunicorn/gunicorn.access.log'
proc_name = 'gunicorn_myblog'
```

使用配置文件启动：  

```bash
cd <项目路径>
gunicorn -c python:RedQueen.gunicorn_conf RedQueen.wsgi:application
```

日志输出：  

```bash
[2018-11-19 23:39:14 +0800] [2930] [INFO] Starting gunicorn 19.9.0
[2018-11-19 23:39:14 +0800] [2930] [INFO] Listening at: unix:/home/cloudsen/work/deploy/webservers/sockets/nginx-gunicorn.sock (2930)
[2018-11-19 23:39:14 +0800] [2930] [INFO] Using worker: gevent
[2018-11-19 23:39:14 +0800] [2933] [INFO] Booting worker with pid: 2933
[2018-11-19 23:39:14 +0800] [2934] [INFO] Booting worker with pid: 2934
[2018-11-19 23:39:14 +0800] [2936] [INFO] Booting worker with pid: 2936
...
```

## 框架设置

由于目前Gunicorn只对Paster应用框架有效，故再次不做讲解。有需要的看官方文档。  



# 设置项清单

这里对Gunicorn的设置项做一个详细的介绍。有些设置只能写在配置文件中，设置的名字就是配置文件中的变量名。

> 在Gunicorn19.7这个新版本，加入了环境变量设置，可以通过 `GUNICORN_CMD_ARGS` 这个环境变量做一个全局配置：  
>
> ```bash
> GUNICORN_CMD_ARGS="--bind=127.0.0.1 --workers=3" gunicorn app:app
> ```



# 线上部署

## 使用虚拟环境

最好在项目的虚拟环境中使用pip安装gunicorn。

记得对Django setting做如下设置：  

```python
# 项目在服务器上部署时，将所有静态文件打包到某个文件夹下
STATIC_ROOT = os.path.join(BASE_DIR, 'collected_statics')
# 不使用django自己的静态资源处理
DEBUG = False
# hosts按需配置
ALLOWED_HOSTS = ['*']
```



## Gunicorn配置文件

gunicorn配置文件写在项目中的 `<项目路径>/RedQueen/RedQueen/gunicorn_conf.py` ：  

```bash
import multiprocessing

bind = 'unix:/home/cloudsen/work/deploy/webservers/sockets/nginx-gunicorn.sock'
# TCP
# bind = '127.0.0.1:8000'
workers = multiprocessing.cpu_count() * 2 + 1
worker_class = 'gevent'
errorlog = '/home/cloudsen/work/deploy/webservers/gunicorn/gunicorn.error.log'
accesslog = '/home/cloudsen/work/deploy/webservers/gunicorn/gunicorn.access.log'
proc_name = 'gunicorn_myblog'
# 启用守护进程，后台运行，默认False
# daemon = True
```

终端启动gunicorn：   

```bash
# 进入虚拟环境
workon red_queen_env
cd work/python/project/RedQueen
gunicorn RedQueen.wsgi:application -c python:RedQueen.gunicorn_conf
```

## Nginx配置

尽管，现在有很多HTTP代理服务器可用，但还是极力推荐使用Nginx。如果你选择了其他代理服务器，请注意，当你使用Gunicorn默认的worers的时候，客户端的缓存将特别慢。没有Nginx做很好的缓存，Gunicorn极易收到拒绝服务(denial-of-service )攻击。  

简单的Nginx配置如下：  

```nginx
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
        server_name localhost 127.0.0.1;
        client_max_body_size 4G;
        keepalive_timeout 5;
        
        # 沒找到网站图标不报错
        location /favicon.ico {
        	access_log off;
        	log_not_found off;
        }
        
        # Nginx处理静态文件
        location /static/ {
            # 使用root最后是在/home/cloudsen/work/python/project/RedQueen/static中获取静态资源
            # root /home/cloudsen/work/python/project/RedQueen;
            # 使用alias最后是在/home/cloudsen/work/python/project/RedQueen/collected_statics/中获取静态资源
            # 我的Django项目执行python manage.py collectstatic指令时，目标文件夹是collected_statics
            alias /home/cloudsen/work/python/project/RedQueen/collected_statics/;
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

终端测试该配置文件的正确性：  

```bash
sudo nginx -t -p ~/work/deploy/webservers/nginx  -c blog_nginx.conf

nginx: the configuration file /home/cloudsen/work/deploy/webservers/nginx/blog_nginx.conf syntax is ok
nginx: configuration file /home/cloudsen/work/deploy/webservers/nginx/blog_nginx.conf test is successful
```

终端使用该配置文件启动nginx服务：  

```bash
sudo nginx -p ~/work/deploy/webservers/nginx  -c blog_nginx.conf
```

访问 `http://127.0.0.1:80 `，页面正常加载，样式脚本正常加载。  

打开nginx的access.log中可以看到：  

```bash
127.0.0.1 - - [20/Nov/2018:00:14:55 +0800] "GET / HTTP/1.1" 301 0 "-" "Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/70.0.3538.102 Safari/537.36"
# nginx将“/”重写为了“/cloudsen_blog”
127.0.0.1 - - [20/Nov/2018:00:14:55 +0800] "GET /cloudsen_blog/ HTTP/1.1" 200 4555 "-" "Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/70.0.3538.102 Safari/537.36"
# 然后下面全是"/static"静态资源的访问
127.0.0.1 - - [20/Nov/2018:00:14:55 +0800] "GET /static/cloudsen_blog/css/base.css HTTP/1.1" 404 571 "http://127.0.0.1/cloudsen_blog/" "Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/70.0.3538.102 Safari/537.36"
...
```

打开gunicorn的access.log中可以看到：  

```bash
# gunicorn没有访问 “/”，也没有访问静态资源文件
- - [20/Nov/2018:00:14:55 +0800] "GET /cloudsen_blog HTTP/1.0" 301 0 "-" "Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/70.0.3538.102 Safari/537.36"
- - [20/Nov/2018:00:14:55 +0800] "GET /cloudsen_blog/ HTTP/1.0" 200 4555 "-" "Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/70.0.3538.102 Safari/537.36"
```

说明Nginx代理并重写了页面服务的http请求，然后通过unix domain socket转发给了Gunicorn，Gunicorn解析http，从Django项目中获取资源。但是呢静态资源由Nginx自己处理了，没有再交给Gunicorn。

# 参考资料

1. [gunicorn官方文档](http://docs.gunicorn.org/en/stable/)

