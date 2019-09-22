[TOC]

# RabbitMQ服务端

## 服务端的安装

### Ubuntu

创建脚本文件 `rabbitmq-install.sh` ：  

```bash
#!/bin/sh

## Install RabbitMQ signing key
curl -fsSL https://github.com/rabbitmq/signing-keys/releases/download/2.0/rabbitmq-release-signing-key.asc | sudo apt-key add -

## Install apt HTTPS transport
sudo apt-get install apt-transport-https

## Add Bintray repositories that provision latest RabbitMQ and Erlang 21.x releases
sudo tee /etc/apt/sources.list.d/bintray.rabbitmq.list <<EOF
deb https://dl.bintray.com/rabbitmq-erlang/debian bionic erlang-21.x
deb https://dl.bintray.com/rabbitmq/debian bionic main
EOF

## Update package indices
sudo apt-get update -y

## Install rabbitmq-server and its dependencies
sudo apt-get install rabbitmq-server -y --fix-missing
```

Ununtu 18对应的是 `Bionic` ，Ubuntu 16对应的是 `Xenial`。  

然后运行该脚本 `sh rabbitmq-install.sh` 即可安装 `rabbitmq-server` 。  



## 运行RabbitMQ服务端

当 `rabbitmq-server` 安装完毕后，会自动通过非特权用户 `rabbitmq` 作为守护进程运行。  

你可以通过 `sudo systemctl start rabbitmq-server` 和 `sudo systemctl stop rabbitmq-server` 来手动控制服务的运行。  



## 配置RabbitMQ

RabbitMQ节点应该可以通过默认值启动，若希望了解更多生产环境的配置请查阅[生产清单](https://www.rabbitmq.com/production-checklist.html)。  

注意：节点是以 `rabbitmq` 用户启动的，若你修改了节点数据库或日志的路径，你需要确保 `rabbitmq` 用户拥有这些路径的权限。  

## 端口访问

为了确保RabbitMQ能正常连接，请确保可以访问以下端口：  

- 4396： `epmd` RabbitMQ和CLI工具使用的节点发现服务；
- 5672,5671：由AMQP 0-9-1和1.0客户端使用，可选TLS；
- 25672：用于节点间、CLI工具间的通讯；
- 35672~35682：CLI工具与节点通信；
- 15672： HTTP API 客户端，管理界面UI，RabbitMQ ADMIN；
- 61613, 61614： 用于[STOMP clients](https://stomp.github.io/stomp-specification-1.2.html)；
- 1883, 8883：用于[MQTT clients](http://mqtt.org/)；
- 15674：用于STOMP-over-WebSockets客户端；
- 15675：用于MQTT-over-WebSockets客户端；

## 默认用户

Broker会创建一个用户名和密码都为 `guest` 的默认用户以供访问。默认情况下，只有通过 `localhost` 来连接Broker时，才会使用这个证书。  

查看[访问控制](https://www.rabbitmq.com/access-control.html)文档，来创建更多的用户，以及删除guest用户。  

## 管理服务

**查看RabbitMQ服务状态：** `sudo systemctl status rabbitmq-server`  

```
● rabbitmq-server.service - RabbitMQ Messaging Server
   Loaded: loaded (/lib/systemd/system/rabbitmq-server.service; enabled
   Active: active (running) since Fri 2019-09-20 05:14:43 EDT; 1 day 20
 Main PID: 26930 (rabbitmq-server)
    Tasks: 88 (limit: 2385)
   CGroup: /system.slice/rabbitmq-server.service
           ├─26930 /bin/sh /usr/sbin/rabbitmq-server
           ├─26938 /bin/sh /usr/lib/rabbitmq/bin/rabbitmq-server
           ├─27108 /usr/lib/erlang/erts-9.2/bin/epmd -daemon
           ├─27223 /usr/lib/erlang/erts-9.2/bin/beam.smp -W w -A 64 -P 
           ├─27333 erl_child_setup 65536
           ├─27397 inet_gethost 4
           └─27398 inet_gethost 4
```

**停止RabbitMQ本地节点：** `sudo systemctl stop rabbitmq-server`  

**开启RabbitMQ本地节点：** `sudo systemctl start rabbitmq-server`  

`rabbitmqctl` 和 `rabbitmq-diagnostics` 之类的CLI工具可以通过 `sudo` 来使用。  

## 日志管理

服务端的日志文件默认存放在 `/var/log/rabbitmq` 路径下。  

`RABBITMQ_LOG_BASE` 可以[重写日志路径](https://www.rabbitmq.com/relocate.html)。  

也可以通过 `sudo journalctl --system | grep rabbitmq` 来查看系统服务的日志：  

```bash
Sep 20 05:14:07 ubuntu sudo[26493]: cloudsen : TTY=pts/0 ; PWD=/home/cloudsen/script ; USER=root ; COMMAND=/usr/bin/tee /etc/apt/sources.list.d/bintray.rabbitmq.list
Sep 20 05:14:07 ubuntu sudo[26497]: cloudsen : TTY=pts/0 ; PWD=/home/cloudsen/script ; USER=root ; COMMAND=/usr/bin/apt-get install rabbitmq-server -y --fix-missing
Sep 20 05:14:37 ubuntu groupadd[26850]: group added to /etc/group: name=rabbitmq, GID=113
Sep 20 05:14:37 ubuntu groupadd[26850]: group added to /etc/gshadow: name=rabbitmq
Sep 20 05:14:37 ubuntu groupadd[26850]: new group: name=rabbitmq, GID=113
Sep 20 05:14:37 ubuntu useradd[26856]: new user: name=rabbitmq, UID=107, GID=113, home=/var/lib/rabbitmq, shell=/usr/sbin/nologin
Sep 20 05:14:37 ubuntu chage[26862]: changed password expiry for rabbitmq
Sep 20 05:14:37 ubuntu chfn[26865]: changed user 'rabbitmq' information
Sep 20 05:14:40 ubuntu rabbitmq[26931]: Waiting for rabbit@ubuntu
Sep 20 05:14:40 ubuntu rabbitmq[26931]: pid is 26938
```

## 日志轮换

Broker会将日志追加到[日志文件](https://www.rabbitmq.com/logging.html)的末尾，它会保留完整的日志历史记录。  

[logrotate](https://linux.die.net/man/8/logrotate)是一个推荐的方式去轮换和压缩日志。默认情况下，`logrotate` 会在 `/var/log/rabbitmq` 路径中每周运行一次。具体的配置见 `/etc/logrotate.d/rabbitmq-server` 。  

## 启用WEB管理界面

首先在终端中执行 `sudo rabbitmq-plugins enable rabbitmq_management` 来启用 `rabbitmq_management` 插件：  

```
The following plugins have been enabled:
  amqp_client
  cowlib
  cowboy
  rabbitmq_web_dispatch
  rabbitmq_management_agent
  rabbitmq_management

Applying plugin configuration to rabbit@ubuntu... started 6 plugins.
```

然后访问 `http://*{node-hostname}*:15672/` 即可看到管理界面，然后本地可以通过默认用户 `guest`，密码 `guest` 登录。  

## 创建远程登陆用户

`guest` 用户只能在localhost登录，因此远程管理还需要自己创建新用户。  

**创建用户**  

```
sudo rabbitmqctl add_user <username> <password>
```

**设置用户为超级管理员**  

```
sudo rabbitmqctl set_user_tags <username> administrator
```

**设置权限**  

```
sudo rabbitmqctl set_permissions -p '/' <username> '.*'<空格>'.*'<空格>'.*'
```

