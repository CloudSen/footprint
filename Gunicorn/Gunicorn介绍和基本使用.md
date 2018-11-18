## Gunicorn介绍

> [官方网站](http://docs.gunicorn.org/en/stable/)

Gunicorn 的名字来源于 `Green Unicorn绿色独角兽` ，它是一个适用于UNIX的Python WSGI HTTP服务器。Gunicorn与各种web框架兼容，部署简单、服务器资源更明确并且速度相当快。默认是同步工作，支持Gevent、Eventlet异步。

### 特点

- 原生支持 `WSGI` , `Django` 与 `Paster`
- 自动化worker进程管理
- 简单的Python配置
- 多worker配置
- 各种服务器钩子，扩展性极强
- 兼容Python 2.x >= 2.6 或者 3.x >= 3.2


## Installation安装

### 安装

通过pip安装：  

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

### 异步的worker

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

  

## 运行Gunicorn

你可以通过命令行的方式运行Gunicorn，也可以嵌入Django或Paster中去使用。  

### 命令行方式

当 `gunicorn` 安装完毕后，就可以使用命令行了。若通过pip安装的，则在虚拟环境中运行；若通过系统安装的，则可以通过终端直接执行。  

#### gunicorn

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

访问http://127.0.0.1:8000就能看到"Hello, World!" 。那么说明gunicorn正常运行。

#### 常用参数

- `-c CONFIG, --config=CONFIG` - 通过以下方式指定一个配置文件，`$(PATH)` 或 `file:$(PATH)` 或 `python:$(MODULE_NAME)` 。
- `-b BIND, --bind=BIND` - 指定要绑定的服务器嵌套字。服务器嵌套字可以用以下格式：`$(HOST)` , `$(HOST):$(POST)` 或者 `unix:$(PATH)` 。一个IP是一个有效的$(HOST)。
- `-w WORKERS, --workers=WORKERS` - 设置worker进程的数量。每个CPU核心能分2~4个worker。
- `-k WORKERCLASS, --worker-class=WORKERCLASS` - worker进程的类型。你能将它设置为 `$(NAME)` ，这个$(NAME)可以是 `sync` , `eventlet` , `gevent` , `tornado` , `gthread` , `gaiohttp(deprecated)` 。`sync` 是默认的，worker同步阻塞运行，效率低。 

- `-n APP_NAME, --name=APP_NAME` - 如果安装了 `setproctitle` python模块，则可以调整Gunicorn进程的名字。

### 集成方式

#### Django

Gunicorn默认查找名为 `application` 的可供WSGI调用的对象。对于典型的Django项目，调用Gunicorn如下所示：  

```bash
gunicorn myproject.wsgi
```

你可以使用 `--env` 来指定Django的配置文件：  

```bash
gunicorn --env DJANGO_SETTINGS_MODULE=myproject.settings myproject.wsgi
```

### 实际操作

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

