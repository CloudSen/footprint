[TOC]

# Python 开发环境

## Python

### 安裝

1. Arch默认的`/usr/bin/python`是链接到 python 3 的，終端键入 `python` 默认就是Python3.7(很爽有木有)，若要使用Python2.7就键入 `python2`；

2. python的路径分别是：`/usr/bin/python3` 和 `/usr/bin/python2` ；

3. 安装 `pip` :

   ```bash
   sudo pacman -S python-pip
   # /usr/bin/pip
   which pip
   # pip 18.0 from /usr/lib/python3.7/site-packages/pip (python 3.7)
   pip -V
   ```

4. 安装虚拟环境 `virtualenv` :

   ```bash
   sudo pip install virtualenv
   ```

5. 安装虚拟环境管理工具 `virtualenvwrapper` :

   ```bash
   sudo pip install virtualenvwrapper
   # 找到它的路径 /usr/bin/virtualenvwrapper.sh
   sudo find / -name virtualenvwrapper.sh
   # 创建好自己的工程文件夹和虚拟环境文件夹
   mkdir ~/work/python/projects
   mkdir ~/work/python/virtual_env
   ```

   配置环境变量，把下面三个变量写入 `.bashrc` 或 `.zshrc` 或 `.profile` ：

   ```shell
   export WORKON_HOME=$HOME/work/python/virtual_env
   export PROJECT_HOME=$HOME/work/python/projects
   source /usr/bin/virtualenvwrapper.sh
   ```

   然后使配置立刻生效（我用的zsh）：

   ```bash
   source ~/.zshrc
   # virtualenvwrapper.user_scripts creating /home/cloudsen/work/python/virtual_env/premkproject
   # virtualenvwrapper.user_scripts creating /home/cloudsen/work/python/virtual_env/postmkproject
   # virtualenvwrapper.user_scripts creating /home/cloudsen/work/python/virtual_env/initialize
   # virtualenvwrapper.user_scripts creating /home/cloudsen/work/python/virtual_env/premkvirtualenv
   # virtualenvwrapper.user_scripts creating /home/cloudsen/work/python/virtual_env/postmkvirtualenv
   # virtualenvwrapper.user_scripts creating /home/cloudsen/work/python/virtual_env/prermvirtualenv
   # virtualenvwrapper.user_scripts creating /home/cloudsen/work/python/virtual_env/postrmvirtualenv
   # virtualenvwrapper.user_scripts creating /home/cloudsen/work/python/virtual_env/predeactivate
   # virtualenvwrapper.user_scripts creating /home/cloudsen/work/python/virtual_env/postdeactivate
   # virtualenvwrapper.user_scripts creating /home/cloudsen/work/python/virtual_env/preactivate
   # virtualenvwrapper.user_scripts creating /home/cloudsen/work/python/virtual_env/postactivate
   # virtualenvwrapper.user_scripts creating /home/cloudsen/work/python/virtual_env/get_env_details
   # 生成一大堆文件
   ```

6. 安装好后查看pip list :

   ```bash
   pip list
   # 发现下面几个包就ok了
   # virtualenv        16.0.0     
   # virtualenv-clone  0.4.0      
   # virtualenvwrapper 4.8.2 
   ```

### 测试

1. 使用 `virtualenv` ：

   ```bash
   cd ~/work/python/virtual_env
   # 创建一个虚拟环境
   virtualenv test
   # >> Using base prefix '/usr'
   # >> New python executable in /home/cloudsen/work/python/virtual_env/test/bin/python
   # >> Installing setuptools, pip, wheel...done.
   # 进入虚拟环境
   source ~/work/python/virtual_env/test/bin/activate
   # >> (test)  cloudsen@GLaDOS  ~/work/python/virtual_env/test/bin  
   # 查看当前虚拟环境的包
   pip list
   # >> pip        18.1   
   # >> setuptools 40.4.3 
   # >> wheel      0.32.1
   # 退出虚拟环境
   deactivate
   ```

2. 使用 `virtualenvwrapper` ：

   > 项目多了后，virtualenv 使用极其不方便，每次都要记住activate的路径；
   >
   > 需要使用wrapper统一管理虚拟环境，切换虚拟环境也非常方便。

   ```bash
   # 创建虚拟环境，test2存放在我们配置的路径下，创建完毕自动进入当前创建的环境中（美滋滋）
   mkvirtualenv test2 
   # >>(test2)  cloudsen@GLaDOS  ~/work/python/virtual_env 
   # 退出当前虚拟环境
   deactivate
   # 列出所有的虚拟环境
   workon
   # >>test1 test2
   # 激活某个虚拟环境
   workon test2
   # 删除虚拟环境
   rmvirtualenv test2
   ```

## pyenv&pyenv-virtualenv

>  pyenv 帮助我们完美的隔离环境，让多个版本的 python 没有任何冲突，完美共存。
>
> 既可以管理系统的各个Python版本，还能管理虚拟环境，一步到位。

**pyenv的作用是管理并隔离多个python版本，virtualenv使用某个版本的Python去创建虚拟环境，从而管理并隔离各种第三方包的安装**。  

### 安装

[安装pyenv](https://github.com/pyenv/pyenv#installation)：  

```bash
git clone https://github.com/pyenv/pyenv.git ~/.pyenv
echo 'export PYENV_ROOT="$HOME/.pyenv"' >> ~/.zshrc
echo 'export PATH="$PYENV_ROOT/bin:$PATH"' >> ~/.zshrc
echo -e 'if command -v pyenv 1>/dev/null 2>&1; then\n  eval "$(pyenv init -)"\nfi' >> ~/.zshrc
source ~/.zshrc
```

安装需要的依赖，[见这里](https://github.com/pyenv/pyenv/wiki)：  

```bash
# Arch Linux
sudo pacman -S base-devel openssl zlib
# Ubuntu/Debian/Mint
sudo apt-get install -y make build-essential libssl-dev zlib1g-dev libbz2-dev libreadline-dev libsqlite3-dev wget curl llvm libncurses5-dev xz-utils tk-dev libxml2-dev libxmlsec1-dev libffi-dev
```

安装完毕后，查看pyenv根目录中的versions文件夹，可以看到没有任何东西：  

```bash
cd ~/.pyenv/versions
ls
```

安装虚拟环境插件：  

```bash
git clone https://github.com/pyenv/pyenv-virtualenv.git $(pyenv root)/plugins/pyenv-virtualenv
# 添加虚拟环境自动激活
echo 'eval "$(pyenv virtualenv-init -)"' >> ~/.zshrc
```

### pyenv使用

查看所有的pyenv指令：  

```bash
pyenv commands
```

通过pyenv安装某个版本的Python：  

```bash
# 查看能够安装的所有版本
pyenv install -l
#　安装指定版本
pyenv install <版本>
# 强行安装，哪怕该版本已安装
pyenv install -f <版本>
# 取消安装，若该版本已安装
pyenv install -s <版本>
```

安装新的python版本后刷新：  

```bash
pyenv rehash
```

查看当前位置版本：  

显示当前激活中的Python版本以及如何设置的信息。

```bash
pyenv version
# 当前使用的系统默认的python版本，由.pyenv/version文件全局配置
system (set by /home/cloudsen/.pyenv/version)
```

查看当前安装的所有Python版本:    

列出当前安装的所有版本，并在当前位置激活的版本前加星号。

```
pyenv versions

* system (set by /home/cloudsen/.pyenv/version)
  3.7.1
```

指定全局版本：  

通过将版本名称写入〜/ .pyenv / version文件，设置要在所有shell中使用的Python的全局版本。它能够被本地版本的 `.python-version` 文件所覆盖，或者被环境变量的 `PYENV_VERSION` 所覆盖。  

```bash

pyenv global xxx
# 指定多个全局python版本，前面的优先
pyenv global xxx1 xxx2 xxx3
```

指定本地版本：  

pyenv可以为某个路径单独设置一个python版本，使用此指令时，会在当前目录创建一个 `.python-version` 文件。本地版本会覆盖全局版本的配置，但也会被 `pyenv shell` 指令设置的终端版本或环境变量中的 `PYENV_VERSION` 覆盖。  

```bash
cd <某个目录>
# 设置当前目录使用python3.7.1
pyenv local 3.7.1
# 执行上面的指令后，生成了.python-version文件
ls -la

drwxr-xr-x 2 cloudsen cloudsen 4096 11月 18 21:53 .
drwxr-xr-x 4 cloudsen cloudsen 4096 11月 18 21:53 ..
-rw-r--r-- 1 cloudsen cloudsen    7 11月 18 21:53 .python-version

文件的内容为：  
cat .python-version
3.7.1
```

当然也可以指定多个本地版本，写在前面的版本优先使用：  

```bash
pyenv local 版本1  版本2
```

指定shell的版本：  

通过在shell中设置 `PYENV_VERSION` 环境变量来设置特定于shell的一个或多个Python版本。此版本会覆盖特定于应用程序的本地版本和全局版本。  

```bash
pyenv shell 版本1  版本2
# 取消设置
pyenv shell --unset
# 也可以手动修改环境变量
export PYENV_VERSION=版本
```

卸载某个已安装的python版本：  

```bash
pyenv uninstall xxx
# 尝试在不提示的情况下删除指定的版本确认。如果版本不存在，请不要显示错误消息。
pyenv uninstall -f xxx
```

展示指令的全路径：  

```bash
pyenv which python
/usr/bin/python
pyenv which pyenv version
/home/cloudsen/.pyenv/libexec/pyenv
pyenv which which
/usr/bin/which
```

更新和卸载 pyenv：  

由于是通过git克隆的项目，没有进行什么安装，卸载特别方便，只需要直接删除在 `~/.zshrc` 中的环境变量和 `~/.pyenv` 目录。更新只需要 `git pull` 。  

### virtualenv插件的使用

与pyenv一起使用：

使用pyenv 的python版本创建虚拟环境，第一个参数指定要使用的python版本，第二个参数指定虚拟环境目录的名称。  

```bash
pyenv virtualenv <python版本> <虚拟环境目录名>
```

如：`pyenv virtualenv 3.7.1 test_virtualenv` 将会创建一个基与 `$(pyenv root)/versions` 目录下的python3.7.1版本名为test_virtualenv的虚拟环境。

从当前python版本创建虚拟环境：  

如果 `pyenv virtualenv` 没有传递python版本参数，则默认使用当前pyenv的python版本。  

```bash
pyenv version
# 可以看到，这里使用的.pyenv/version全局配置
3.4.3 (set by /home/cloudsen/.pyenv/version)
pyenv virtualenv <虚拟环境目录名>
```

列出所有已存在的虚拟环境(包括conda的)：  

```bash
pyenv virtualenvs
miniconda3-3.9.1 (created from /home/yyuu/.pyenv/versions/miniconda3-3.9.1)
miniconda3-3.9.1/envs/myenv (created from /home/yyuu/.pyenv/versions/miniconda3-3.9.1)
2.7.10/envs/my-virtual-env-2.7.10 (created from /home/yyuu/.pyenv/versions/2.7.10)
3.4.3/envs/venv34 (created from /home/yyuu/.pyenv/versions/3.4.3)
my-virtual-env-2.7.10 (created from /home/yyuu/.pyenv/versions/2.7.10)
* venv34 (created from /home/yyuu/.pyenv/versions/3.4.3)
```

可以看到，每个虚拟环境有两个入口，一个名称较长在某个python版本下的envs文件夹中；一个名字很短，是一个符号链接。  

激活/失效虚拟环境：  

如果在shell中已经配置了 `eval "$(pyenv virtualenv-init -)` 的话，并且 `.python-version` 文件中指定了 `pyenv virtualenvs` 的某个虚拟环境，那么在你进出这个文件夹的时候，会自动的激活/失效当前虚拟环境。  

当然你也可以手动激活/失效：  

```bash
pyenv activate <虚拟环境名>
pyenv deactivate
```

删除存在的虚拟环境：   

你可以手动删除这两个文件来移除某个虚拟环境： `$(pyenv root)/versions` 和 `$(pyenv root)/versions/{version}/envs` 。或者使用便捷指令：  

```bash
pyenv uninstall <虚拟环境名>
```

virtualenv 和 venv：  

CPython 3.3和之后的python版本有一个名叫 `venv` 的模块。它提供了一个可执行模块venv，是virtualenv的继承者。  

如果venv是可用的且 `virtualenv` 不可用，则 `pyenv-virtualenv` 使用 `python -m venv` 。  

### 实际演示

安装一个版本为3.8-dev的python，创建一个项目文件夹 `test` ，然后为这个项目创建虚拟环境，并指定该项目文件夹使用创建的虚拟环境：  

```bash
mkdir ~/Documents/test

pyenv install -l
pyenv install 3.8-dev
pyenv versions
    * system (set by /home/cloudsen/.pyenv/version)
      3.8-dev

pyenv virtualenv 3.8-dev test-env
    Looking in links: /tmp/tmpfrj5ikv0
    Requirement already satisfied: setuptools in /home/cloudsen/.pyenv/versions/3.8-dev/envs/test-env/lib/python3.8/site-packages (39.0.1)
    Requirement already satisfied: pip in /home/cloudsen/.pyenv/versions/3.8-dev/envs/test-env/lib/python3.8/site-packages (10.0.1)
pyenv virtualenvs
    3.8-dev/envs/test-env (created from /home/cloudsen/.pyenv/versions/3.8-dev)
    test-env (created from /home/cloudsen/.pyenv/versions/3.8-dev)

cd ~/Documents/test
pyenv local test-env
ls -la
	drwxr-xr-x 2 cloudsen cloudsen 4096 11月 19 00:08 .
    drwxr-xr-x 4 cloudsen cloudsen 4096 11月 18 21:53 ..
    -rw-r--r-- 1 cloudsen cloudsen    8 11月 19 00:08 .python-version
pyenv version
	test-env (set by /home/cloudsen/Documents/test/.python-version)

# 进入目录自动激活虚拟环境
cd ..
cd ./test/
	(test-env)  cloudsen@GLaDOS ~/Documents/test：
pyenv versions
	system
	3.8-dev
	3.8-dev/envs/test-env
	* test-env (set by /home/cloudsen/Documents/test/.python-version)

# 退出目录自动失效虚拟环境
cd ..
pyenv versions
	* system (set by /home/cloudsen/.pyenv/version)
	3.8-dev
	3.8-dev/envs/test-env
	test-env
```



## Pycharm

#### 安装

去官网直接下载压缩包，然后解压到制定目录：

```bash
tar -zxvf pycharm-professional-2018.2.4.tar.gz -C ~/soft/pycharm
rm -rf tar -zxvf pycharm-professional-2018.2.4.tar.gz
```

然后就可以启动 `pycharm` 了：

```bash
sh ~/soft/pycharm/pycharm-2018.2.4/bin/pycharm.sh
```

`pycharm` 激活之后，可以生成一个 `lanuncher` python脚本，执行它能便捷启动应用程序。

#### 测试

不想手动 `sh` 启动pycharm，又不想手动执行 `lanuncher` 脚本，怎么办？

那就生成一个创建快捷方式。

方法一：

```bash
sudo ln -s ~/soft/pycharm/pycharm-2018.2.4/bin/pycharm.sh /usr/bin/pycharm
```

方法二：

在 `/usr/share/applications` 创建一个 `.desktop` 文件。

```bash
sudo vim /usr/share/applications/pycharm.desktop  
```

写入以下内容：

```text
[Desktop Entry]
Encoding=UTF-8 
Name=pycharm-professional 
Comment=IDE for python
Exec=/home/cloudsen/soft/pycharm/pycharm-2018.2.4/bin/pycharm.sh
Icon=pycharm-professional
Terminal=false 
StartupNotify=true 
Type=Application
Categories=Application;Development;
```



  

# Java 开发环境

> 使用Orical的JAVASE，不适用OpenJDK，免得遇到奇奇怪怪的问题。

## JDK

### 安装

Oracle 的 `JDK` 和 `JRE` 都能在AUR中下载，非常方便：  

```bash
# 记住选Oracle的JDK
# aur/jdk9 9.0.4-1 (+7 0.77%) 
# Oracle Java 9 Development Kit (public release - end of suppor
yay jdk9
yay jdk8
yay jdk7
```

### 测试

```bash
# 查看系統安裝的所有JAVA
# Available Java environments:
# java-8-jdk
# java-9-jdk (default)
archlinux-java status
# 改变系统默认的JDK
sudo archlinux-java set java-8-jdk
# 取消系统所有默认JDK
sudo archlinux-java unset
# 修复默认的JDK环境
sudo archlinux-java fix

# java version "9.0.4"
# Java(TM) SE Runtime Environment (build 9.0.4+11)
# Java HotSpot(TM) 64-Bit Server VM (build 9.0.4+11, mixed mode)
 java -version
 # javac 9.0.4
 javac -version
# /usr/share/java
# /usr/bin/java
# /usr/lib/jvm/java-9-jdk/bin/java
# /home/cloudsen/soft/pycharm/pycharm-2018.2.4/jre64/bin/java
# /etc/ssl/certs/java
sudo find / -name java
```

## Maven

### 安装

去官网下载压缩包：[Download](https://maven.apache.org/download.cgi) ，然后解压缩到指定目录。  

在 `.zshrc` 文件中加入以下內容：  

```bash
# java
export JAVA_HOME=/usr/lib/jvm/java-8-jdk
export MAVEN_HOME=/home/cloudsen/soft/apache-maven-3.6.0
export PATH="$MAVEN_HOME/bin:$PATH"
```

在终端中查看maven版本，若有以下信息，则说明安装完毕，可以正常使用：  

```
$ mvn -v
Apache Maven 3.6.0 (97c98ec64a1fdfee7767ce5ffb20918da4f719f3; 2018-10-25T02:41:47+08:00)
Maven home: /home/cloudsen/soft/apache-maven-3.6.0
Java version: 1.8.0_192, vendor: Oracle Corporation, runtime: /usr/lib/jvm/java-8-jdk/jre
Default locale: zh_CN, platform encoding: UTF-8
OS name: "linux", version: "4.19.2-arch1-1-arch", arch: "amd64", family: "unix"

```

### 配置

编辑 `$MAVEN_HOME/conf/` 路径下的 `settings.xml` 文件，主要设置一下本地仓库的位置：  

```xml
<localRepository>/home/cloudsen/work/java/mvnRepository/</localRepository>
```



# JetBrains 实用插件

## .ignore

生成许多项目类型的git忽略文件配置。

## CodeGlance

在右侧显示一列小小的代码minmap，方便定位。

## Background Image Plus

设置编辑器的背景图片。

## Material Theme UI

material质感设计主题，现在加入了详细的设置界面，非常棒。

## ESLint

纠正JavaScript的代码语法错误和规范错误

## Pylint

纠正Python的代码语法错误和规范错误  



# Mysql

> 详细说明见 [Arch Wiki Mysql](https://wiki.archlinux.org/index.php/MySQL)

Arch Linux , Debian, CentOS等开发版本已经去掉了Oracle官方的Mysql，使用的Mysql开源分支版本MariaDB。MariaDB兼容Mysql。  

## 安装

Arch Linux应该已经预装了MariaDB，但是需要以ROOT身份初始化：  

```bash
mysql_install_db --user=mysql --basedir=/usr --datadir=/var/lib/mysql
```

启动 `mariadb` [守护进程](https://wiki.archlinux.org/index.php/Daemon)，运行安装脚本，然后重新启动守护进程：  

```bash
systemctl start mariadb
mysql_secure_installation
systemctl restart mariadb
```

运行安装脚本时，mysql root密码默认为空，然后需要重新设置mysql root密码等操作。  

## 配置

### 添加新用戶

访问域一般为localhost 或者 %，根据情况给用户分配数据库使用权。

```bash
mysql -u root -p
-----------------
MariaDB> CREATE USER '<用戶名>'@'<访问域>' IDENTIFIED BY '<密码>';
MariaDB> GRANT ALL PRIVILEGES ON <某个数据库名>.<表名> TO '<用戶名>'@'<访问域>'
MariaDB> FLUSH PRIVILEGES;
MariaDB> quit
```

### 配置文件

*MariaDB* 会按照以下顺序读取配置文件 (根据 `mysqld --help --verbose` 的输出):  

```bash
/etc/my.cnf /etc/mysql/my.cnf ~/.my.cnf
```

### 禁用远程访问

如果只有本机需要 MySQL，可以通过不监听 TCP 端口 3306 来增强安全性。要拒绝远程连接，取消注释 `/etc/mysql/my.cnf` 中以下这行：  

```
skip-networking
```

### 启用自动补全

编辑 `/etc/mysql/my.cnf`，将 `no-auto-rehash` 替换为 `auto-rehash`。下次客户端启动时就会启用自动补全。  

### 使用UTF-8编码

在 `/etc/mysql/my.cnf` 的 `mysqld` 下, 添加:

```
[mysqld]
init_connect                = 'SET collation_connection = utf8_general_ci,NAMES utf8'
collation_server            = utf8_general_ci
character_set_client        = utf8
character_set_server        = utf8
```

### 备份

数据库可以转储到文件以简化备份。以下 shell 脚本会替你在脚本所在目录创建一个 `db_backup.gz` 文件，包含数据库的转储：

```
#!/bin/bash

THISDIR=$(dirname $(readlink -f "$0"))

mysqldump --single-transaction --flush-logs --master-data=2 --all-databases \
 | gzip > $THISDIR/db_backup.gz
echo 'purge master logs before date_sub(now(), interval 7 day);' | mysql
```

### 限制 logfile 的大小

默认情况下， mysqld 会在 `/var/lib/mysql` 下创建二进制日志文件。可以在 `/etc/mysql/my.cnf` 中修改以下內容限制日志大小：  

```bash
expire_logs_days = 10
max_binlog_size  = 100M
```

### dbeaver客戶端

安裝：  

```bash
yay dbeaver
```

