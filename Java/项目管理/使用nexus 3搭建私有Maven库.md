[TOC]

# 前言

## 预准备

- 支持Systemd的Linux发行版（本文采用Arch Linux，其他发行版需自行替换相关命令）
- 30分钟+

## 将获得

- 使用传统安装方式，安装、配置并启动nexus 3 仓库
- 使用shell脚本，批量上传本地包到私有库
- 配置Maven Settings文件，让项目下载私有库的包
- 配置项目pom文件，让项目打包部署到私有库



# 下载

去[官网](https://help.sonatype.com/repomanager3/download)下载最新的压缩包。 

 

# 安装

将压缩包解压到`/opt`目录下：  

```bash
sudo tar zxf ~/Downloads/nexus-<你的版本号>-unix.tar.gz  -C /opt && rm -f ~/Downloads/nexus-<你的版本号>-unix.tar.gz
```



# 配置

## 修改数据保存路径

编辑`<安装路径>/bin/nexus.vmoptions`文件即可修改数据储存的路径，如：  

```
-Dkaraf.data=/home/clouds3n/Downloads/nexus/karaf
-Djava.io.tmpdir=/home/clouds3n/Downloads/nexus/tmp
-XX:LogFile=/home/clouds3n/Downloads/nexus/jvm-log/jvm.log
-Dkaraf.log=/home/clouds3n/Downloads/nexus/log
```

## 修改HTTP端口

默认的http端口是8081，因此默认只需要访问`http://localhost:8081`即可访问nexus页面。nexus在启动后，会在数据路径自动生成`$data-dir/etc/nexus.properties`配置文件，可以更改里面的`application-port`来修改HTTP端口。  



# 通过Systemd的方式启动

在`/etc/systemd/system/`目录下创建nexus的服务文件：  

```
sudo vim /etc/systemd/system/nexus.service
```

然后写入以下内容（注意修改版本号）：  

```
[Unit]
Description=nexus service
After=network.target
  
[Service]nexus
Type=forking
LimitNOFILE=65536
ExecStart=/opt/nexus-<你的版本号>/bin/nexus start
ExecStop=/opt/nexus-<你的版本号>/bin/nexus stop
User=root
Restart=on-abort
TimeoutSec=600
  
[Install]
WantedBy=multi-user.target
```

重新加载系统服务：  

```
sudo systemctl daemon-reload
```

设置开机自启动（可选）：  

```
sudo systemctl enable nexus.service
```

启动服务：  

```
sudo systemctl start nexus.service
```

停止服务：  

```
sudo systemctl stop nexus.service
```

重启服务：  

```
sudo systemctl restart nexus.service
```

查看服务状态：  

```
systemctl status nexus
```

查看nexus日志（默认路径）：  

```
tail -f /opt/sonatype-work/nexus3/log/nexus.log
```



# 访问页面

若没有修改端口，则直接进入`http://localhost:8081`即可。  

登录的时候，管理员密码存放在`-Dkaraf.data`指定目录的`admin.password`文件中（页面有提示）。输入后既可登录。  

登录之后，根据页面提示进行配置。  



# 脚本上传本地库到私有库

首先在页面创建自己的私有maven库，例如我创建了一个名叫`common-repo`的maven(hosted)私有库。  

然后将以下脚本文件放到本地maven库目录下：  

**deploy.sh**  

```
#!/bin/bash

while getopts ":u:p:r:" opt; do
    case $opt in
        r)
            echo "r: $OPTARG"
            REPO_URL="$OPTARG"
            ;;
        u)
            echo "u: $OPTARG"
            USERNAME="$OPTARG"
            ;;
        p)
            echo "p: $OPTARG"
            PASSWORD="$OPTARG"
            ;;
        *)
            echo "$opt not exists!"
            ;;
    esac
done

find . -type f \
-not -path './mavenimport\.sh*' \
-not -path '*/\.*' \
-not -path '*/\^archetype\-catalog\.xml*' \
-not -path '*/\^maven\-metadata\-local*\.xml' \
-not -path '*/\^maven\-metadata\-deployment*\.xml' \
-not -path '*/_remote*' \
-not -path '*/*\.lastUpdated*' \
| sed "s|^\./||" | xargs -I '{}' \
curl -u "$USERNAME:$PASSWORD" -X PUT -v -T {} ${REPO_URL}/{} ;
```

然后赋予脚本可执行权限：  `chmod +x deploy.sh`  

执行脚本，传入参数，即可开始批量上传本地库。  

```
./deploy.sh -u admin -p cloudsen -r http://localhost:8081/repository/common-repo/
```



# 项目使用私有厍

> maven仓库的优先级如下：
>
> 本地仓库 > 私服（profile）> 远程仓库（repository）

按照下方的内容，修改本地maven `settings.xml`配置文件。该文件指定了中央仓库为阿里云，配置了nexus私服地址，以及本地仓库地址。  

**settings.xml**  

```
<?xml version="1.0" encoding="UTF-8"?>
<settings xmlns="http://maven.apache.org/SETTINGS/1.0.0"
          xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
          xsi:schemaLocation="http://maven.apache.org/SETTINGS/1.0.0 http://maven.apache.org/xsd/settings-1.0.0.xsd">
  
  <localRepository>/home/clouds3n/Downloads/mvn_repo</localRepository>
  
  <pluginGroups>
  </pluginGroups>

  <proxies>
  </proxies>

  <servers>
  </servers>

  <mirrors>
     <mirror>
      <id>aliyunmaven</id>
      <mirrorOf>central</mirrorOf>
      <name>阿里云公共仓库</name>
      <url>https://maven.aliyun.com/repository/public</url>
    </mirror>
  </mirrors>

  <profiles>
    <profile>
      <id>local-nexus</id>
      <repositories>
        <repository>
            <id>nexus</id>
            <url>http://localhost:8081/repository/common-repo/</url>
            <releases>
                <enabled>true</enabled>
            </releases>
            <snapshots>
                <enabled>true</enabled>
            </snapshots>
        </repository>
      </repositories>
      <pluginRepositories>
          <pluginRepository>
            <id>nexus</id>
            <url>http://localhost:8081/repository/common-repo/</url>
            <releases>
                <enabled>true</enabled>
            </releases>
            <snapshots>
                <enabled>true</enabled>
            </snapshots>
          </pluginRepository>
      </pluginRepositories>
    </profile>
  </profiles>

  <activeProfiles>
      <activeProfile>local-nexus</activeProfile>
  </activeProfiles>
</settings>
```



# 项目打包部署到私有库

在需要部署的项目`pom.xml`文件中加入以下配置：  

```
<distributionManagement>
  <repository>
    <id>nexus</id>
    <name>common-repo</name>
    <url>http://localhost:8081/repository/common-repo/</url>
  </repository>
</distributionManagement>
```

然后在maven `settings.xml`文件的`servers`标签中加入以下配置（注意id要与pom文件中的对应）：  

```
<server>
	<id>nexus</id>
	<username>admin</username>
	<password>cloudsen</password>
</server>
```

最后执行maven的`deploy`命令即可部署到私有库。  



# 结语

既然，我们可以从中央仓库下载资源，为什么还要搭建私有库呢？最主要的原因有以下几点：  

- 节省外网带宽

  一个人使用中央库没问题，但当公司有几十上百人一起使用中央库下载资源的时候，是十分占用外网资源的。因此，公司需要避免多人长时间下载这样的操作，搭建私有库能很好解决这个问题。

- 内网环境开发

  在一些要求严格的公司里，是不允许使用外网的，因此外部的中央仓库、远程仓库都无法使用。我们也不可能在手动安装jar包到本地maven库，这太麻烦了，因此需要内网的私有仓库。

- 发布私有包

  如果你想分享自己的包给大家使用，需要上传到中央仓库中。但是有些公司不允许公开内部包，此时还想分享给公司其他人使用，只能上传到私有仓库。

- 并不仅仅是maven

  nexus并不仅仅只能作为mven仓库使用，它还可以搭建npm、docker等私有库，功能十分强大。