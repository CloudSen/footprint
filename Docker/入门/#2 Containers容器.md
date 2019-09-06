[TOC]

# Containers 容器

## 介绍

是时候以Docker的方式构建应用了。我们从下面的目录结构开始介绍，本章节主要介绍 `container` 容器。容器的上一层是 `service` 服务，它定义了各个容器的行为，服务在第3章节中介绍。最顶层是 `stack` 栈，它定义了所有服务的交互，栈在第5章节中介绍。  

- `Stack`
- `Services` 
- `Container` （你所在的位置）

## 新的开发环境

通常，当你开始开发一个Python应用时，你第一个任务是在你的机器上安装Python运行环境。但是这样会产生一个问题：本地开发环境和线上环境需要高度保持一致。  

当使用Docker后，你仅需抓取一份可移植的Python运行环境镜像，不需要安装。然后你的构建可以包含基础的Python镜像以及你的应用代码，确保你的应用、依赖、运行环境都在一起。  

这些可移植的镜像都被一个称为 `Dockerfile` 的文件定义。  

## 通过Dockerfile定义容器

`Dockerfile` 文件定义了容器中的环境。在这个环境中，网络和硬盘访问都被虚拟化了，因此容器的环境与系统的其他部分是完全隔离的。因此，你需要将环境中的端口和外部系统进行映射，并说明要“复制”到该环境的文件。因此，凡是应用此 `Dockerfile` 来定义的应用，在任何地方运行时的结果都是一致的。  

**Dockerfile**  

在本地创建一个空的目录，然后在该目录中创建一个名为 `Dockerfile` 的文件。将以下内容粘贴到此文件中：  

```dockerfile
# Use an official Python runtime as a parent image
FROM python:2.7-slim

# Set the working directory to /app
WORKDIR /app

# Copy the current directory contents into the container at /app
COPY . /app

# Install any needed packages specified in requirements.txt
RUN pip install --trusted-host pypi.python.org -r requirements.txt

# Make port 80 available to the world outside this container
EXPOSE 80

# Define environment variable
ENV NAME World

# Run app.py when the container launches
CMD ["python", "app.py"]
```

然后再在该目录中继续创建 `requirements.txt` 和 `app.py` 两个文件。当根据 `Dockerfile` 构建镜像时，这两个文件就会被包含进去，因为在 `Dockerfile` 中我们进行了复制操作。  

**requirements.txt**



