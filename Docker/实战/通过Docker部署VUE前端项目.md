[TOC]

# 通过Docker部署VUE前端项目

## 打包前端项目

```bash
npm install
npm run build
```

得到打包后的 `dist` 文件夹，将它复制到指定的位置，我的路径是 `/home/cloudsen/work/deploy/webapps/blog-vue-front/dist`。  

## 拉取Nginx镜像

```bash
sudo docker pull nginx

Using default tag: latest
latest: Pulling from library/nginx
1ab2bdfe9778: Pull complete
a17e64cfe253: Pull complete
e1288088c7a8: Pull complete
Digest: sha256:1a8935aae56694cee3090d39df51b4e7fcbfe6877df24a4c5c0782dfeccc97e1
Status: Downloaded newer image for nginx:latest
docker.io/library/nginx:latest
```

查看拉取的Nginx镜像：  

```bash
sudo docker images

REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
nginx               latest              5a3221f0137b        3 weeks ago         126MB
```

## 编写Nginx配置文件

在 `/home/cloudsen/work/deploy/webapps/blog-vue-front` 目录下创建nginx配置文件 `vue-nginx.config`，用于替代Nginx的 `default.conf`：  

```nginx
server {
    # 如果没有HOST匹配，则关闭链接防止HOST欺骗
    listen 80 default_server;
    return 444;
}

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

    ssl on;
    ssl_certificate /usr/share/nginx/ssl/yangyunsen.com.pem;
    ssl_certificate_key /usr/share/nginx/ssl/yangyunsen.com.key;
    ssl_session_cache shared:SSL:10m;
    error_page 497 https://www.yanagyunsen.com;

    root /usr/share/nginx/html/; 
    index index.html;

    location /favicon.ico {
        alias /usr/share/nginx/html/;
        access_log off;
        log_not_found off;
    }

    location / {
        try_files $uri $uri/ @router;
        index index.html;
    }

    location @router {
        rewrite ^.*$ /index.html last;
    }
} 

```

## 编写Dockerfile

在在 `/home/cloudsen/work/deploy/webapps/blog-vue-front` 目录下创建 `Dockerfile` 文件：  

```dockerfile
FROM nginx
RUN mkdir -p /usr/share/nginx/ssl/
COPY yangyunsen.com.pem /usr/share/nginx/ssl/yangyunsen.com.pem
COPY yangyunsen.com.key /usr/share/nginx/ssl/yangyunsen.com.key
COPY dist/ /usr/share/nginx/html/ 
COPY vue-nginx.conf /etc/nginx/conf.d/default.conf 
```

- 定义了自定义镜像基于官方nginx镜像；
- 在容器中创建了一个ssl目录，用于存放ssl证书；
- 将前端打包好的静态目录，复制到容器中的 `/usr/share/nginx/html/`；
- 将自定义的nginx默认配置，复制到容器中的 `/etc/nginx/conf.d/default.conf`；

## 生成自定义镜像

`/home/cloudsen/work/deploy/webapps/blog-vue-front` 路径结构如下：  

```
dist/
vue-nginx.conf
Dockerfile
yangyunsen.com.key
yangyunsen.com.pem
```

通过自己编写的 `Dockerfile` 生成新的自定义镜像，该镜像继承与官方nginx，但是使用的是自定义的配置：  

```bash
cd /home/cloudsen/work/deploy/dockerImages/blog-vue-front
sudo docker build -t blog-vue-nginx .
```

```bash
Sending build context to Docker daemon  4.504MB
Step 1/6 : FROM nginx
 ---> 5a3221f0137b
Step 2/6 : RUN mkdir -p /usr/share/nginx/ssl/
 ---> Running in 189147665bfc
Removing intermediate container 189147665bfc
 ---> 929489b804bf
Step 3/6 : COPY yangyunsen.com.pem /usr/share/nginx/ssl/yangyunsen.com.pem
 ---> a33947b064f6
Step 4/6 : COPY yangyunsen.com.key /usr/share/nginx/ssl/yangyunsen.com.key
 ---> cc8b35fcf6ca
Step 5/6 : COPY dist/ /usr/share/nginx/html/
 ---> 632378450350
Step 6/6 : COPY vue-nginx.conf /etc/nginx/conf.d/default.conf
 ---> 22a82ae11148
Successfully built 22a82ae11148
Successfully tagged blog-vue-nginx:latest
```

然后就可以看见镜像列表中出现了 `blog-vue-nginx` ：  

```bash
sudo docker images
```

```bash
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
blog-vue-nginx      latest              22a82ae11148        21 seconds ago      130MB
nginx               latest              5a3221f0137b        5 weeks ago         126MB
```

##  启动容器

通过以下指令将自定义镜像生成容器，并启动容器：  

```bash
docker run -d \
-p 80:80 \
-p 443:443 \
--name blog-vue-container \
blog-vue-nginx
```

- `-d` 表示后台运行；
- ssl需要443端口，`-p` 将宿主机的80端口和443端口，映射到了容器的80和443端口
- `--name` 设置容器名字为blog-vue-container；

查看运行中的容器：  

```bash
sudo docker ps
```

访问域名，测试是否能够访问。  





