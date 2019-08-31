[TOC]

# Docker相关概念和环境配置

## Docker介绍

`Docker` 是开发人员和系统管理人员通过容器 `Container` 来开发、部署和运行应用程序的一个平台。使用Linux容器去部署应用程序这个过程叫做容器化 `Containerization`。容器并不是一个新颖的概念，但是它能让我们部署应用更加方便。  

目前，容器化越来越受欢迎，因为它有以下优点：  

- Flexible灵活：就算是最复杂的应用，也可以被容器化；
- Lightweight轻量级：容器之间共享主机 `Host` 内核；
- Interchangeable拥抱变化：可以即时部署更新和升级；
- Portable可移植：你可以在本地构建，然后部署在云端，从而在任何地方运行；
- Scalable可扩展：你可以新增或自动发布容器副本；
- Stackable可堆叠：你可以即时垂直堆叠服务。

### 镜像Image和容器Container

- **镜像**：一个镜像就是一个可执行的包，它包含了你的应用程序运行时所需要的所有文件（代码、运行环境、依赖、环境变量、配置文件等等）。
- **容器**：是镜像运行时的实例（镜像执行后在内存中的部分），你可以把容器类比为用户进程，通过 `docker ps` 可以列出所有正在运行的容器。

**所以，它们的关系是：通过运行镜像来启动容器，容器是镜像的实例。**

### 容器和虚拟机

容器仅仅包含了某个应用运行期间所需得基本资源，容器自己没有操作系统内核，容器之间共享宿主系统内核，非常轻量化。  

而虚拟机中不但包含了某个应用运行期间所需得基本资源，自己还有操作系统内核以及一切杂七杂八的冗余资源，每个虚拟机都要消耗大量资源，部署成本高。  

![container_vm](https://s2.ax1x.com/2019/08/29/mbjv2n.png)  



## 准备你的Docker环境

见另一篇文章：Docker社区版安装和测试。  



## 本章小节

```bash
# 帮助
## 列出所有的Docker命令
docker
## 列出所有的container命令
docker container --help

# 显示
## 显示Docker版本和更为详细的版本信息
docker --version
sudo docker version
## 显示Docker的信息
sudo docker info

# 镜像
## 运行镜像
sudo docker run <镜像名>
## 列出已有的镜像
sudo docker image ls

#容器
### 列出正在运行中的容器
sudo docker container ls
### 列出所有的容器
sudo docker container ls --all
### 列出所有的容器，仅显示ID
sudo docker container ls -aq
```

