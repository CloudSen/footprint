[TOC]

# 概要

## 你将获得

在Linux环境(Arch&Ubuntu)中，成功安装Docker，并运行HelloWorld镜像。

## 预准备

- 通畅的网络
- Arch 或 Ubuntu系统



# Docker社区版安装和测试

Docker社区引擎的定位是为小型团队或个人开发者提供入门体验。它有三种版本：stable，test和nightly：  

- Stable稳定版：提供最新的一般功能版本；
- Test测试版：提供Stable之前的预发布版本；
- Nightly每夜版：提供最新的开发特性版本；

## 支持的平台

桌面系统支持：MacOS和Windows。  

服务端支持：CentOS、Debian、Fedora、Ubuntu、Raspbian

你也可以下载[二进制文件](https://docs.docker.com/install/linux/docker-ce/binaries/)手动安装到自己喜欢的Linux版本。  

## 安装Docker

> 默认使用的存储驱动为 `overlay2`

### Arch Linux安装Docker

> 参考：[Arch Wiki - Docker](https://wiki.archlinux.org/index.php/Docker)。
>
> WARN: 我使用的yay包管理，具体见Arch Linux日常环境搭建。

**安装:**  

```bash
yay -S docker
```

执行完毕后，就会安装这四个包：`bridge-utils`,`containerd` ,`runc` ,`docker` 。

**启动Docker服务，并设置开机启动：**  

> 注意：先把代理软件关闭再执行！

```bash
sudo systemctl start docker
sudo systemctl enable docker
```

### Ubuntu安装Docker

**添加仓库：**  

```bash
sudo apt-get update
```

```bash
sudo apt-get install \
    apt-transport-https \
    ca-certificates \
    curl \
    gnupg-agent \
    software-properties-common
```

```bash
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
```

验证是否存在key拥有指纹 `9DC8 5822 9FC7 DD38 854A E2D8 8D81 803C 0EBF CD88` ：  

```bash
sudo apt-key fingerprint 0EBFCD88

=> 
pub   rsa4096 2017-02-22 [SCEA]
      9DC8 5822 9FC7 DD38 854A  E2D8 8D81 803C 0EBF CD88
uid           [ unknown] Docker Release (CE deb) <docker@docker.com>
sub   rsa4096 2017-02-22 [S]
```

通过以下的指令指定 `stable` 仓库，若你希望添加其他仓库，则把 `stable` 改为 `test` 或 `nightly` 即可：  

```bash
sudo add-apt-repository \
   "deb [arch=amd64] https://download.docker.com/linux/ubuntu \
   $(lsb_release -cs) \
   stable"
```

>`lsb_release -cs` 子命令会返回当前Ubuntu的发布版本名（如，xenial）。  
>
>如果你使用的诸如 `Linux Mint` 这样的Ubuntu衍生系统，你需要将上面的 `$(lsb_release -cs)` 手动替换为对应的Ubuntu版本，例如，你使用的 `Linux Mint Tessa` 则此处应该替换为 `bionic`。

**安装：**  

```
sudo apt-get update
sudo apt-get install docker-ce docker-ce-cli containerd.io
```

## 测试安装是否成功

### 检查Docker版本

```bash
docker --version
=>
Docker version 19.03.1-ce, build 74b1e89e8a

sudo docker version
=>
Client:
 Version:           19.03.1-ce
 API version:       1.40
 Go version:        go1.12.8
 Git commit:        74b1e89e8a
 Built:             Fri Aug 16 14:00:42 2019
 OS/Arch:           linux/amd64
 Experimental:      false

Server:
 Engine:
  Version:          19.03.1-ce
  API version:      1.40 (minimum version 1.12)
  Go version:       go1.12.8
  Git commit:       74b1e89e8a
  Built:            Fri Aug 16 13:58:02 2019
  OS/Arch:          linux/amd64
  Experimental:     false
 containerd:
  Version:          v1.2.8.m
  GitCommit:        a4bc1d432a2c33aa2eed37f338dceabb93641310.m
 runc:
  Version:          1.0.0-rc8
  GitCommit:        425e105d5a03fabd737a126ad93d62a9eeede87f
 docker-init:
  Version:          0.18.0
  GitCommit:        fec3683
```

### 查看Docker详细信息

```
sudo docker info
```

### 检查Docker是否正常使用

```bash
sudo docker run hello-world

=>
Unable to find image 'hello-world:latest' locally
latest: Pulling from library/hello-world
1b930d010525: Pull complete 
Digest: sha256:451ce787d12369c5df2a32c85e5a03d52cbcef6eb3586dd03075f3034f10adcd
Status: Downloaded newer image for hello-world:latest

Hello from Docker!
This message shows that your installation appears to be working correctly.

To generate this message, Docker took the following steps:
 1. The Docker client contacted the Docker daemon.
 2. The Docker daemon pulled the "hello-world" image from the Docker Hub.
    (amd64)
 3. The Docker daemon created a new container from that image which runs the
    executable that produces the output you are currently reading.
 4. The Docker daemon streamed that output to the Docker client, which sent it
    to your terminal.

To try something more ambitious, you can run an Ubuntu container with:
 $ docker run -it ubuntu bash

Share images, automate workflows, and more with a free Docker ID:
 https://hub.docker.com/

For more examples and ideas, visit:
 https://docs.docker.com/get-started/

```

执行以上指令后，docker会运行hello-world镜像，由于本地没有这个镜像，因此它会去DockerHub上拉取最新的该镜像，并运行，然后生成一个对应的容器。  

**列出已有的镜像列表：**  

使用 `docker image ls` 或者 `docker images` ：  

```bash
sudo docker image ls

REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
hello-world         latest              fce289e99eb9        8 months ago        1.84kB
```

**列出已有的容器列表：**  

```bash
sudo docker container ls

=>
CONTAINER ID   IMAGE          COMMAND     CREATED          STATUS     ...
9853d39541eb   hello-world    "/hello"    24 minutes ago   Exited (0) ...
```

到此，说明已成功安装Docker，并运行了一个测试镜像。  



# 参考

- 官方文档：[Install Docker Engine](https://docs.docker.com/engine/install/)

