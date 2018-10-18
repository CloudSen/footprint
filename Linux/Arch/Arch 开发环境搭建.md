# Arch 开发环境搭建

  

## Python 开发环境

### Python

#### 安裝

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

#### 测试

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

### Pycharm

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



  

## Java 开发环境

> 使用Orical的JAVASE，不适用OpenJDK，免得遇到奇奇怪怪的问题。

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

  

## JetBrains 实用插件

### .ignore

生成许多项目类型的git忽略文件配置。

### CodeGlance

在右侧显示一列小小的代码minmap，方便定位。

### Background Image Plus

设置编辑器的背景图片。

### Material Theme UI

material质感设计主题，现在加入了详细的设置界面，非常棒。

### ESLint

纠正JavaScript的代码语法错误和规范错误

### Pylint

纠正Python的代码语法错误和规范错误