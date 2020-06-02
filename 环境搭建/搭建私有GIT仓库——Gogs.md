[TOC]

# 概要

## 你将获得

在Linux环境中，以gogs用户，手动二进制安装Gogs、配置Gogs并运行。

## 预准备

> Mysql必须使用INNODB引擎、utf8_general_ci charset编码格式

- 支持Systemd的Linux发行版
- Linux上备有MySQL、MariaDB、PostgreSQL、MSSQL、SQLite3 或 TiDB中的任意一个数据库
- Linux上已安装Git
- Gogs安装包，[点击下载](https://dl.gogs.io/)



# 开始安装

## 新建用户环境

新建gogs用户，分配密码，并创建用户目录：  

```
# useradd gogs -m -s /bin/bash -p <PASSWORD>
```

将用户gogs添加到gogs用户组：  

```
usermod -a -G gogs gogs
```

## 准备安装文件

**切换到gogs用户**，并在用户目录下创建soft文件夹，然后上传Gogs安装包到此地：  

```text
# su gogs
$ mkdir -p ~/soft && cd ~/soft
```

解压zip压缩包:  

```
$ unzip -q gogs_0.11.91_linux_amd64.zip && rm -f gogs_0.11.91_linux_amd64.zip
```

解压完成后，便得到gogs的软件目录。  

## 启动Gogs

**切换回root用户**，将Gogs提供的Systemd自启脚本拷贝到系统中：  

```
# cp /home/gogs/soft/gogs/scripts/systemd/gogs.service /etc/systemd/system/
```

编辑 `gogs.service` :  

```
# vi /etc/systemd/system/gogs.service
```

替换为以下内容：  

```
[Unit]
Description=Gogs
After=syslog.target
After=network.target
After=mariadb.service mysqld.service postgresql.service memcached.service redis.service

[Service]
# Modify these two values and uncomment them if you have
# repos with lots of files and get an HTTP error 500 because
# of that
###
LimitMEMLOCK=infinity
LimitNOFILE=65535
Type=simple
User=gogs
Group=gogs
WorkingDirectory=/home/gogs/soft/gogs
ExecStart=/home/gogs/soft/gogs/gogs web
Restart=always

# Some distributions may not support these hardening directives. If you cannot start the service due
# to an unknown option, comment out the ones not supported by your version of systemd.
ProtectSystem=full
PrivateDevices=yes
PrivateTmp=yes
NoNewPrivileges=true

[Install]
WantedBy=multi-user.target
```

通过 `systemctl` 启动gogs，并设置gogs开机自启：  

```
# systemctl daemon-reload
# systemctl start gogs
# systemctl enable gogs
```

查看gogs服务状态：  

```
# systemctl status gogs -l
```

如果看到日志中出现监听3000端口，说明已经启动成功：  

```
...
[ INFO] Listen: http://0.0.0.0:3000
```

## 配置Gogs

关闭防火墙（此处可以选择开放相应的端口）：  

```
# systemctl stop firewalld
```

打开浏览器，访问 `http://localhost:3000` 进入图形化界面。此时，你需要填写三大配置：  

- 数据库配置
  - Database Type：根据需要选择
  - Host：数据库地址
  - User：数据库用户名
  - Password：数据库密码
  - Database Name：gogs使用的数据库名
  - Path: SQLite3文件地址，使用绝对路径
- 应用配置
  - App Name：取个牛逼的名字
  - Repo Root Path：Git仓库存放地址
  - Run User：gogs
  - Domain：服务器ip地址
  - SSH Port：默认22
  - HTTP Port：默认3000
  - Application URL：http://[Domain]:[HTTP Port]/
  - Log Path：日志存放地址
- 可选配置

例如，我现在使用SQLite3作为Gogs的数据库，那么数据库配置为：  

- Database Type：SQLite3
- Path：/home/gogs/data/gogs.db

应用配置为：  

- Repo Root Path：/home/gogs/gogs-repositories
- Run User: gogs

## 使用Gogs

配置完毕后，即可进入登陆界面，注册的第一个账号既是超级管理员。  



# Well Done~!

到此，私有Git仓库已经搭建完毕，Good Luck~ Have Fun~ :)