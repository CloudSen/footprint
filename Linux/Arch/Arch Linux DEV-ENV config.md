# Arch Linux WEB开发环境搭建

[TOC]

## 网络工具

安装网络工具集合：  

```
yay net-tools
```



## 数据库环境

### Mariadb

> 参考[Arch Wiki Mariadb](https://wiki.archlinux.org/index.php/MariaDB#Installation)

**安装**  

```
yay -S mariadb
```

**配置**

```
sudo mariadb-install-db --user=mysql --basedir=/usr --datadir=/var/lib/mysql
```

**启动**  

```
systemctl start mariadb
systemctl enable mariadb
```

**安全配置（可选）**  

```
sudo mysql_secure_installation
```

**添加用户**  

先使用root用户登录，默认密码为空：  

```
sudo mysql -u root -p
```

添加用户，并分配权限：  

```
CREATE USER 'clouds3n'@'*' IDENTIFIED BY 'cloudsen';
GRANT ALL PRIVILEGES ON *.* TO 'clouds3n'@'*';
FLUSH PRIVILEGES;
```

**安装Mysql-Workbench**  

```
yay -S mysql-workbench
```





## 前端环境

前端环境列表：  

- NodeJS
- VUE-CLI3
- Nginx

### NodeJS

**安装：**  

NodeJS核心：  

```
yay -Syy nodejs
```

npm包管理：  

```
yay -Syy npm
```

**配置：**  

修改全局安装地址：

```
npm config set prefix "path"
```

修改缓存地址：  

```
npm config set cache "path"
```

淘宝镜像：  

```
npm install -g cnpm --registry=https://registry.npm.taobao.org
```

配置cnpm软链接：  

```
sudo ln -s <全局安装绝对路径>/package/lib/node_modules/cnpm/bin/cnpm /usr/local/bin/cnpm
```

**检查是否安装成功：**  

```
node --version
npm --version
```

###  VUE-CLI3脚手架

> [官方文档](https://cli.vuejs.org/zh/guide/#%E8%AF%A5%E7%B3%BB%E7%BB%9F%E7%9A%84%E7%BB%84%E4%BB%B6)

**安装：  **

```
npm i -g @vue/cli
```

**配置：**  

```
sudo ln -s <npm包绝对路径>/vue.js /usr/local/bin/vue
```

**检查脚手架版本：**    

```
vue -V
```



## JAVA环境

JAVA环境列表：  

- JDK8&JDK11

### JDK8&JDK11

**安装：**  

```
yay java-8-openjdk java-8-openjdk
```

**检查系统所有JDK版本：**  

```
archlinux-java status
```

**查看当前默认使用的JDK：**  

```
archlinux-java get
```

**切换系统默认JDK：**    

```
archlinux-java set <java_env>
```



## 打包&部署环境

- DOCKER
- MAVEN
- GRADLE

### Docker

**安装：**  

```
yay -Syy Docker
```

**启动：**  

```
systemctl enable docker
systemctl start docker
systemctl status docker
```

**检查版本：**  

```
docker -v
docker info
```

### Maven

**安装：**  

去[官网](https://maven.apache.org/download.cgi)下载tar包，然后解压到自定义目录：  

```
tar zxf xxx.tar.gz
```

**配置：**  

在`.zshrc`文件中加入以下内容：  

```
# java
export JAVA_HOME=/usr/lib/jvm/java-8-jdk
export MAVEN_HOME=/home/cloudsen/soft/apache-maven-3.6.0
export PATH="$MAVEN_HOME/bin:$PATH"
```

**检查版本：**  

```
mvn -v
```



## DOCKER环境

