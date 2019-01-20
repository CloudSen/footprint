[TOC]

# Nginx Web Server #1 将Nginx配置为WEB服务器

在顶层，将Nginx配置为一个WEB服务器的关键在于明确一些URLs，以及这些URLs如何去处理对资源的HTTP请求。在底层，配置文件定义了很多虚拟服务集 `server` ，它们控制并处理特定域或IP地址的请求。  

用于HTTP流量的每个虚拟服务定义了一些称为 `localtion` 的配置实例，这些实例用于处理一些特定的URIs集。每个 `localtion` 都定义了一些自己的场景去说明映射到该 `localtion` 的请求发生了什么。每个 `location` 还能够代理请求或者返回文件。另外，为了使请求可以被重定向到另一个 `location` 或 `server` ，URI是可以被修改的。当然，`location` 还能返回特定的错误码，你甚至还可以为每个错误码指定对应的页面。  

## 配置虚拟服务

在配置文件中必须包含至少一个 `server` 指令来定义虚拟服务。当Nginx处理请求时，它首先会选择请求对应的虚拟服务。  

在 `http` 上下文中通过 `server` 指令定义一个或多个虚拟服务：  

```Nginx config files
http {
    server {
        # 虚拟服务1配置
    }
    
    server {
        # 虚拟服务2配置
    }
}
```

`server` 配置块通常包含一个 `listen` 指令，用于指定服务器侦听请求的IP地址和端口（或Unix域套接字和路径）。IPv4和IPv6都是支持的，若使用IPv6地址则需要方括号括起来。  

下面的例子展示了一个监听127.0.01地址和8080端口的服务配置：  

```Nginx config files
server {
    listen 127.0.0.1:80
    # 其他配置
}
```

如果端口是缺省的，则使用标准端口。同样，如果IP地址缺省，则监听所有地址。如果 `listen` 指令不存在，则“标准”端口是 `80/tcp` ，“默认端口”是 `8000/tcp` ，具体取决于超级用户权限。  

如果有多个 `server` 与请求的IP地址和端口匹配，Nginx会通过 `Host头` 来与 `server` 块中的 `server_name` 指令比对。`server_name` 字段的参数可以是全名、通配符 `*` 、正则表达式。Nginx的正则表达式使用Perl语法，使用 `~` 波浪号代表正则开始。下面的例子展示了一个全名：  

```Nginx config files
server {
    listen      80; # 监听所有IP的80端口
    server_name example.org www.example.org; # 匹配HOST头为这两个的地址
}
```







*[虚拟服务]: Virtual Servers  

*[HTML]: Hyper Text Markup Language  