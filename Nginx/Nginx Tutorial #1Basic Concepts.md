# Nginx指南 #2概念与基础



## 什么是Nginx？

> C10K：web2.0初期，不能很好地去解决每秒上万次的并发请求，只有靠堆硬件勉强硬撑。现在面对的问题是C10M，每秒千万级别的并发请求。
>
> 上游服务器(upstream servers)：上游，有源头的意思，即代表tomcat、uWSGI之类的产生内容的服务器。

Nginx是一款高性能、轻量级的web和反向代理服务器，它以事件驱动的方式编写。

Nginx最初是为了解决[C10K问题](https://en.wikipedia.org/wiki/C10k_problem)而创造的产物。首先它可以为一个WEB服务器，快速地为你的数据服务。其次你可以使用它实现反向代理(reverse proxy)，并轻易的集成一些如`Unicorn` 和 `Puma` 这样相对缓慢的上游服务器。当然，你还可以用它实现负载均衡(load balancing)，合理分流、动态调整图片、内容缓存、IMAP、POP3、SMTP等等。

Nginx最基本的结构是由master进程和多个相互独立的worker进程组成。master会读取配置文件以及管理workers，workers会处理requests请求。  



## Directive指令和Context上下文

Nginx的配置文件根据系统的不同存放在以下路径：

- /etc/nginx/nginx.conf
- /user/local/etc/nginx/nginx.conf
- /user/local/nginx/conf/nginx.conf

配置文件由Directive和Context組成:  

- Directive: 由名字和参数构成，以分号结尾。

    ```
    gzip on;
    ```

- Context:  是一个你能声明Directive的块，类似编程语言的作用域，使用花括号。

    ```
    worker_processes 2; # directive in global context
    gzip off; # directive in global context
    
    http {              # http context
        gzip on;        # directive in http context
    
      server {          # server context
        listen 80;      # directive in server context
      }
    }
    ```

### Directive 类型

因为不同Directive的**继承模型**不同，因此当你在不同的Context中使用相同的Directive时，必须格外小心。以下是一些便于理解的Directive类型。  

#### Normal普通类型

每个Context中只能定义一次且只有一个值。Subcontext能重写父类的Directive，但重写的有效性只在当前Subcontext中有效。  

```
gzip on;
gzip off; # 在同一个Context中非法定义2个相同的普通Directive

server {
  location /downloads {
    gzip off; # Subcontext重写Directive
  }

  location /assets {
    # 这里继承的全局gzip
  }
}
```

#### Array 数组类型

在同一个Context中定义多个相同的Directive，这些Directive的值会进行累加，而不是被覆盖。若在Subcontext中重写这个Directive，那么将会覆盖父类中的所有值。重写的有效性只在当前Subcontext中有效。  

```
error_log /var/log/nginx/error.log;
error_log /var/log/nginx/error_notive.log notice;
error_log /var/log/nginx/error_debug.log debug;

server {
  location /downloads {
    # 这里将会重写父类所有的 directives
    error_log /var/log/nginx/error_downloads.log;
  }
}
```

#### Action 动作类型

我们可以将一些改变事物的指令叫做action directive。它们的继承行为根据模块而不同。  

例如，`rewrite` directive，每个匹配directive都会被执行：  

```
server {
    rewrite ^ /foobar;
    
    location /foobar {
        rewrite ^ /foo;
        rewirte ^ /bar;
    }
}
```

如果用户尝试获取 `/sample` 的资源，上面例子的执行过程如下：  

- 首先 `server` 上下文中的 `rewrite` 被执行，将 `/sample` 重写为 `/foobar`
- 然后 `location` 上下文匹配到 `/foobar`
- 接着 `location` 中的第一个 `rewrite` 被执行，将 `/foobar` 重写为 `/foo`
- 最后 `location` 中的第二个 `rewrite` 被执行，将 `foo` 重写为 `/bar`

这与 `return` 指定的行为是不同的：  

```
server {
    location / {
        return 200;
        return 404;
    }
}
```

如上示例，`200` 状态码会被立即返回，不会执行第二个 `return`。



## 处理请求

在nginx中，你可以定义多个通过 `server {}` 定义的虚拟服务。  

```
server {
  listen      *:80 default_server;
  server_name netguru.co;

  return 200 "Hello from netguru.co";
}

server {
  listen      *:80;
  server_name foo.co;

  return 200 "Hello from foo.co";
}

server {
  listen      *:81;
  server_name bar.co;

  return 200 "Hello from bar.co";
}
```

```
Request to foo.co:80     => "Hello from foo.co"
Request to www.foo.co:80 => "Hello from netguru.co"
Request to bar.co:80     => "Hello from netguru.co"
Request to bar.co:81     => "Hello from bar.co"
Request to foo.co:81     => "Hello from bar.co"
```

如上示例，提供了nginx如何处理请求的配置。nginx首先会检查 `listen` 指令，以此来匹配哪个 `server` 正在监听当前请求的 IP:port组合，上面的例子会监听所有IP的端口。然后 `server_name` 指令将会匹配当前请求的服务器域名。  

总之Nginx对虚拟服务的选择优先级如下：  

1. 有匹配的IP:Port监听，匹配到对应的 `server_name` ；
2. 有匹配的IP:Port监听，匹配到定义了 `default_server` 的默认服务；
3. 有匹配的IP:Port监听，匹配第一个定义的服务；
4. 以上都匹配失败，拒绝当前连接。

### server_name 指令

`server_name` 指令允许设置多个值，它还能能匹配通配符和正则表达式。  

```
server_name netguru.co www.netguru.co; # 精确匹配
server_name *.netguru.co;              # 通配符匹配
server_name netguru.*;                 # 通配符匹配
server_name  ~^[0-9]*\.netguru\.co$;   # 正则匹配
```

当有歧义的时候，`server_name` 使用下面的优先级：  

1.  确切的名字；
2.  以 `*` 开头的，最长的通配符名，e.g. “*.example.org”;
3.  以 `*` 结尾的，最长的通配符名，e.g. “mail.*”；
4.  根据在配置文件中的顺序，第一个与正则表达式匹配的；

Nginx会存储3个哈希表：精确名，以 `*` 开头的通配名，以 `*` 结尾的通配名。如果在这三张表中都没找到结果，那么正则表达式就会被根据顺序执行。  

值得牢记于心的是  

```
server_name .netguru.co
```

它是以下的缩写：  

```
server_name netguru.co www.netguru.co *.netguru.co
```

但这两种写法有点不同的是：`.netguru.co` 被存储在第二个表中，这意味着在速度上，它比精确描述的写法稍慢。

### listen 指令

在大多数情况下，你会发现 `listen` 指令存贮IP:port值。  

```
listen 127.0.0.1:80;
listen 127.0.0.1;    # 默认使用80端口

listen *:81;
listen 81;           # 默认使用所有IP

listen [::]:80;      # IPv6 addresses
listen [::1];        # IPv6 addresses
```

当然，它也可以指定 UNIX-domain sockets：  

```
listen unix:/var/run/nginx.sock;
```

你还可以直接使用主机名：  

```
listen localhost:80;
listen netguru.co:80;
```

这应该谨慎使用，因为在nginx启动时可能无法解析主机名，导致nginx无法在给定的TCP socket上绑定。  

最后，如果 `listen` 不存在，那么默认使用 `*:80` 。  



## 最小配置

具备以上知识后，我们就可以创建并理解运行Nginx的最小配置。  

```
# /etc/nginx/nginx.conf

events {} # 事件处理

http {
 server {
    listen 80;
    server_name  netguru.co  www.netguru.co  *.netguru.co;

    return 200 "Hello";
  }
}
```

#### root 指令

 `root` 指令设置了请求的资源根目录，它允许nginx将请求映射到文件系统上。  

```
server {
  listen 80;
  server_name netguru.co;
  root /var/www/netguru.co;
}
```

它允许nginx根据请求返回服务器内容：  

```
netguru.co:80/index.html     # returns /var/www/netguru.co/index.html
netguru.co:80/foo/index.html # returns /var/www/netguru.co/foo/index.html
```

#### location 指令

`location` 指令根据请求的URI进行设置。语法是 `location [modifier] path`。  

```
localtion /foo {
    # 你的配置...
}
```

如果没有修饰符，path将作为前缀对待，在它后面可以追加任何内容，以上的示例可以匹配：  

```
/foo
/fooo
/foo123
/foo/bar/index.html
...
```

在同一个上下文中，可以使用多个 `location` 指令：  

```
events {}

http {
    server {
        listen 80;
        server_name localhost default_server;
        root ~/work/python/project/RedQueen/cloudsen_blog/templates/cloudsen_blog;
        
        location / {
            return 200 "root path";
        }
        
        location /home {
            return 200 "home page";
        }
    }
}
```

```
localhost:80/  =>  "root path"
localhost:80/home  =>  "home page"
localhost:80/homepage  =>  "home page"
ocalhost:80/nomatch  =>  "root path"
```

Nginx还提供了一些能够与 `location` 结合使用的修饰符。

```
=           - 精确匹配
^~          - 优先匹配
~ && ~*     - 正则表达式匹配，~*大小写不敏感，~大小写敏感
no modifier - 没有使用修饰符或以上三个都没匹配到，将path当作前缀进行匹配
```

```
location /match {
  return 200 'Prefix match: matches everything that starting with /match';
}

location ~* /match[0-9] {
  return 200 'Case insensitive regex match';
}

location ~ /MATCH[0-9] {
  return 200 'Case sensitive regex match';
}

location ^~ /match0 {
  return 200 'Preferential match';
}

location = /match {
  return 200 'Exact match';
}
```

```
/match      => 'Exact match'
/match0     => 'Preferential match'
/match1     => 'Case insensitive regex match'
/MATCH1     => 'Case sensitive regex match'
/match-abc  => 'Prefix match: matches everything that starting with /match'
```

#### try_files 指令

`try_files` 指令将尝试不同的路径，返回找到的任何一个。  

```
try_files $uri index.html =404;
```

例如 `/foo.html` ，它会通过以下优先级返回文件：  

1.  `$uri` ，这里是 `/foo.html`；
2.  `index.html`；
3.  若没有找到任何一个，返回404。

有趣的是，如果我们在 `server` 上下文中定义了 `try_files` ，然后又定义了能匹配所有请求的 `location` ，那么 `try_files` 不会被执行。因为在 `server` 上下文中的 `try_files` 定义了一个自己的伪位置，这个伪位置是最不具体的位置，优先级非常低，其他的 `location` 比这个伪位置更加具体。

```
server {
  try_files $uri /index.html =404; # 不会执行

  location / {
  }
}
```

因此，要避免在 `server` 上下文中定义 `try_files` 。  

 

## 版权信息

本文大量参考他人成果以及官方文档，本人进行了翻译和归纳总结，原作者信息如下：

作者：Mateusz Dobek

链接：[Nginx Tutorial #1: Basic Concepts](https://www.netguru.co/codestories/nginx-tutorial-basics-concepts)

发布于：netguru