[TOC]

# 安装

之前的文章已经Gunicorn做了详细介绍，请自行查阅。上一步安装项目依赖的时候，已经在虚拟环境中安装Gunicorn以及gevent异步worker类型：  

```bash
cd ~/work/deploy/webapps/RedQueen
pip list
	gevent 1.3.7 
	gunicorn 19.9.0
```



# 修改Django项目的settings

注意以下这两项内容：  

```bash
# 项目在服务器上部署时，将所有静态文件打包到某个文件夹下
STATIC_ROOT = os.path.join(BASE_DIR, 'collected_statics')
# 不使用django自己的静态资源处理
DEBUG = False
```

# 创建相关路径

```bash
mkdir -p ~/work/deploy/webservers/gunicorn/
mkdir -p ~/work/deploy/webservers/sockets/
```

# Gunicorn配置文件

gunicorn配置文件写在项目中的 `<项目路径>/RedQueen/RedQueen/gunicorn_conf.py` ：  

```bash
import multiprocessing

bind = 'unix:/home/cloudsen/work/deploy/webservers/sockets/nginx-gunicorn.sock'
#bind = '127.0.0.1:8000'
workers = multiprocessing.cpu_count() * 2 + 1
worker_class = 'gevent'
errorlog = '/home/cloudsen/work/deploy/webservers/gunicorn/gunicorn.error.log'
accesslog = '/home/cloudsen/work/deploy/webservers/gunicorn/gunicorn.access.log'
proc_name = 'gunicorn_myblog'
```

# 后台启动Gunicorn

```bash
cd ~/work/deploy/webapps/RedQueen
gunicorn RedQueen.wsgi:application -c python:RedQueen.gunicorn_conf -D
```

  

查看 `/home/cloudsen/work/deploy/webservers/gunicorn/gunicorn.error.log` 日志：  

```
[2018-11-19 14:39:15 -0500] [9509] [INFO] Starting gunicorn 19.9.0
[2018-11-19 14:39:15 -0500] [9509] [INFO] Listening at: unix:/home/cloudsen/work/deploy/webservers/sockets/nginx-gunicorn.sock (9509)
[2018-11-19 14:39:15 -0500] [9509] [INFO] Using worker: gevent
[2018-11-19 14:39:15 -0500] [9550] [INFO] Booting worker with pid: 9550
[2018-11-19 14:39:15 -0500] [9551] [INFO] Booting worker with pid: 9551
[2018-11-19 14:39:16 -0500] [9552] [INFO] Booting worker with pid: 9552
[2018-11-19 14:39:16 -0500] [9553] [INFO] Booting worker with pid: 9553
[2018-11-19 14:39:16 -0500] [9554] [INFO] Booting worker with pid: 9554
```

若没有报错，则启动成功！接下来就差Nginx了！  

# 关闭Gunicorn

```bash
apt-get install psmisc
pstree -ap |grep gunicorn

|-gunicorn,2196 /home/cloudsen/.pyenv/versions/red_queen_env/bin/gunicornRedQuee
  |   |-gunicorn,2244 /home/cloudsen/.pyenv/versions/red_queen_env/bin/gunicornRedQuee
  |   |-gunicorn,2249 /home/cloudsen/.pyenv/versions/red_queen_env/bin/gunicornRedQuee
  |   |-gunicorn,2251 /home/cloudsen/.pyenv/versions/red_queen_env/bin/gunicornRedQuee
  |   |-gunicorn,2262 /home/cloudsen/.pyenv/versions/red_queen_env/bin/gunicornRedQuee
  |   `-gunicorn,2263 /home/cloudsen/.pyenv/versions/red_queen_env/bin/gunicornRedQuee
  
kill -9 2196
```

