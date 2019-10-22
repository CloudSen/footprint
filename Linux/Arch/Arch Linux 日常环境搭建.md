[TOC]

# Arch Linux 日常环境搭建

![background](https://s1.ax1x.com/2018/12/19/FDy5xx.png)  



## 换国内源

镜像源：  

```bash
cp /etc/pacman.d/mirrorlist /etc/pacman.d/mirrorlist.backup

vim /etc/pacman.d/mirrorlist
```

```
## Aliyun
Server = http://mirrors.aliyun.com/archlinux/$repo/os/$arch
## qing hua
Server = https://mirrors.tuna.tsinghua.edu.cn/archlinux/$repo/os/$arch
## USTC
Server = https://mirrors.ustc.edu.cn/archlinux/$repo/os/$arch
## shang hai jiao tong
Server = https://mirrors.sjtug.sjtu.edu.cn/archlinux/$repo/os/$arch
```

社区仓库源：  

```bash
cp /etc/pacman.conf /etc/pacman.conf.backup

vim /etc/pacman.conf
```

追加：  

```
[archlinuxcn]
# The Chinese Arch Linux communities packages.
# SigLevel = Optional TrustedOnly
SigLevel = Optional TrustAll
# 清华大学
Server = https://mirrors.tuna.tsinghua.edu.cn/archlinuxcn/$arch
```

AUR源：  

```bash
yay --aururl https://aur.tuna.tsinghua.edu.cn --save
```

查看当前配置：  

```bash
yay -P -g
```

```
{
        "aururl": "https://aur.tuna.tsinghua.edu.cn",
        "buildDir": "/home/cloudsen/.cache/yay",
        ......
}
```





## ZSH

大名鼎鼎的 `ZSH` 就不解释了。主要是安装后面的 `oh-my-zsh`

```bash
sudo pacman -S zsh
# 查看zsh是否安装完毕
zsh --version
# 将bash切换为zsh
# 可以直接编辑passwd文件
sudo vim /etc/passwd
# 也可以这样
chsh -s $(which zsh)
```

  

## robbyrussell/oh-my-zsh

`oh-my-zsh` 是用来管理 `zsh` 配置的框架，`zsh` 的默认配置及其复杂繁琐，而 `oh-my-zsh` 让 `zsh` 配置降到0门槛。而且它完全兼容 `bash`，并且提供了很多使用的插件和美观的主题。  

### 安装

```bash
sh -c "$(curl -fsSL https://raw.github.com/robbyrussell/oh-my-zsh/master/tools/install.sh)"
```

![oh](https://s1.ax1x.com/2018/12/19/FDyqde.png)  

安装完毕后，在 `~` 目录生成新的 `.zshrc` 配置文件。  

若有乱码，则安装powerline字体库：

```bash
cd ~/Downloads/
git clone https://github.com/powerline/fonts.git --depth=1
cd fonts/
./install.sh
cd ..
rm -rf fonts
```

### 插件与主题

官方自带插件存放在：`~/.oh-my-zsh/plugins/  `

第三方插件存放在：`~/.oh-my-zsh/custom/plugins/`  

官方自带主题存放在：`~/.oh-my-zsh/themes/`  

第三方主题存放在:  `~/.oh-my-zsh/custom/themes/`  

安装 [spaceship](https://github.com/denysdovhan/spaceship-prompt) 主题：  

```bash
git clone https://github.com/denysdovhan/spaceship-prompt.git "$ZSH_CUSTOM/themes/spaceship-prompt"
```

```bash
ln -s "$ZSH_CUSTOM/themes/spaceship-prompt/spaceship.zsh-theme" "$ZSH_CUSTOM/themes/spaceship.zsh-theme"
```

### 配置

修改 `.zshrc` ，加入以下内容，还有很多插件后期再介绍：  

```bash
# 修改主题
ZSH_THEME="spaceship"

# Useful aliases
alias zshconfig="mate ~/.zshrc"
alias ohmyzsh="mate ~/.oh-my-zsh"
alias getIp="curl cip.cc"
# Terminal Proxy
alias enable_proxy="export ALL_PROXY=socks5h://127.0.0.1:1080"
alias disable_proxy="unset ALL_PROXY"
# VNC remote control
alias start_vnc="vncserver -geometry 1366x768 -alwaysshared -depth 24 -dpi 96 :1"
alias stop_vnc="vncserver -kill :1"

# Ibus
export GTK_IM_MODULE=ibus
export XMODIFIERS=@im=ibus
export QT_IM_MODULE=ibus
ibus-daemon -drx

# Maven
export JAVA_HOME=/usr/lib/jvm/java-8-jdk
export MAVEN_HOME=/home/cloudsen/Download/apache-maven-3.6.2/
export PATH="$MAVEN_HOME/bin:$PATH"


# Python virtualenv wrapper
export WORKON_HOME=$HOME/work/python/virtual_env
export PROJECT_HOME=$HOME/work/python/projects
source /usr/bin/virtualenvwrapper.sh

```

![omz](https://s1.ax1x.com/2018/12/19/FDyTsK.png)  

  

## Yakuake

下拉式终端模拟，嗯，very nice。默认 `F12` 弹出下拉框。

```bash
pacman -S yakuake
```

![yakuake](https://s1.ax1x.com/2018/12/19/FDy7qO.png)  

  点击这里下载[Yakuake-Material主题](https://store.kde.org/p/1229144/)，下载完毕后，进入Yakuake `Configure Yakuake->Appearance` 安装主题。  

![29](https://s1.ax1x.com/2018/12/19/FDyXid.png)  



## Xmind 8

> Xmind是java开发的，必须安装Java环境。见 [Arch Linux 开发环境配置]()。
>
> emmm。。。。不知道为什么，Ubuntu上能打开文件名有中文的文件，Arch上就只能打开纯英文命名的文件。。。
>
> 请支持正版软件，学生购买有很大优惠，就一份steam游戏的钱。

画思维导图必备。  

直接从官网下载 `.zip` 压缩包，解压到指定目录，然后执行 `/<解压路径>/xmind-8-update8-linux/XMind_amd64` 下面的XMind就可以运行了：  

```bash
cd ~/soft/xmind-8-update8-linux/XMind_amd64
./XMind
```

注意！这里有个大坑！JDK8以上打开时报错，配置 `XMind_amd64/XMind.ini` ，在末尾追加以下内容：

```text
--add-modules=ALL-SYSTEM
--illegal-access=warn
```

一定要注意！若系统默认java环境是JDK9，则会抛出ClassNotFoundException，换成JDK8或者修改 `XMind.ini` 就没报错，大概日志如下：  

```java
...
Caused by: java.lang.NoClassDefFoundError: javax/annotation/PostConstruct
	at org.eclipse.e4.core.internal.di.InjectorImpl.inject(InjectorImpl.java:151)
	at org.eclipse.e4.core.internal.di.InjectorImpl.internalMake(InjectorImpl.java:375)
	... 23 more
Caused by: java.lang.ClassNotFoundException: javax.annotation.PostConstruct cannot be found by org.eclipse.e4.core.di_1.6.0.v20160319-0612
	at org.eclipse.osgi.internal.loader.BundleLoader.findClassInternal(BundleLoader.java:398)
	at org.eclipse.osgi.internal.loader.BundleLoader.findClass(BundleLoader.java:361)
	at org.eclipse.osgi.internal.loader.BundleLoader.findClass(BundleLoader.java:353)
	at org.eclipse.osgi.internal.loader.ModuleClassLoader.loadClass(ModuleClassLoader.java:161)
	at java.base/java.lang.ClassLoader.loadClass(ClassLoader.java:496)
	... 25 more
	...
```

新建 `.desktop` 文件，写入以下内容，就能创建快捷方式：

```bash
[Desktop Entry]
Encoding=UTF-8
Name=XMind
Comment=xmind update 8
Exec=/home/cloudsen/soft/xmind-8-update8-linux/xmind.sh
Icon=/home/cloudsen/soft/xmind-8-update8-linux/XMind_icon.png
Terminal=false
StartupNotify=true
Type=Application
Categories=Application;Development;
```

新建启动脚本 `vim ~/soft/xmind-8-update8-linux/xmind.sh` 写入以下内容：

```bash
#!/bin/bash
cd /home/cloudsen/soft/xmind-8-update8-linux/XMind_amd64/
/home/cloudsen/soft/xmind-8-update8-linux/XMind_amd64/XMind
```



## yay

> Yet Another Yogurt - An AUR Helper Written in Go

`yaourt` 已经停更了， [yay](https://github.com/Jguer/yay) 是一款非常不错的AUR安装神器，使用 `GO` 语言编写而成，相当于将 `pacman` 进行了二次包装。  

### 安装

```bash
git clone https://aur.archlinux.org/yay.git
cd yay
makepkg -si
```

### 配置

彩色输出：将 `/etc/pacman.conf` 文件中 `Color` 的注释去掉。  

### 使用

1. 替代 `pacman` 少打几个字母：

   ```bash
   yay -S <包名>
   # 等同于
   pacman -S <包名>
   ```

2. 搜索包，并选择性安装：

   ```bash
   yay <包名>
   ```

   ![yay1](https://s1.ax1x.com/2018/12/19/FDygZF.png)  

3. 打印系统状态：

   ```bash
   yay -Ps
   ```

   ![yay2](https://s1.ax1x.com/2018/12/19/FDyUaQ.png)  

4. 清除不需要的依赖包：

   ```bash
   yay -Yc
   ```

   ![yay3](https://s1.ax1x.com/2018/12/19/FDya5j.png)  

5. 检查并更新系统和安装的软件：

   ```bash
   yay -Syu
   ```

   ![yay4](https://s1.ax1x.com/2018/12/19/FDywPs.png)  

  

## Typora

![ty1](https://s1.ax1x.com/2018/12/19/FDyB2q.png)  

MarkDown编辑神器，所见即所得。

```bash
yay -S typora
```

  

## 网易云音乐

```bash
# 官方的
yay -S netease-cloud-music
# 可以搜索一下AUR库，有electron版本和终端版的
yay netease-cloud-music
```

  

## 系统代理

### ShadowSocksR

> 你懂的socks5代理，推荐使用跨平台的electron-ssr，图形界面和订阅很方便。

```bash
yay electron-ssr
```

![ssr](https://s1.ax1x.com/2018/12/19/FDyfi9.png)  

### 终端使用代理

编辑 `~/.zshrc` ，加入以下 `alias` ：

```text
alias getip="curl -i http://ip.cn"
alias setproxy="export ALL_PROXY=socks5://127.0.0.1:1080"
alias unsetproxy="unset ALL_PROXY"
```

然后更新设置：

```bash
source ~/.zshrc
```

需要代理的时候就 `setproxy` ，不需要的时候 `unsetproxy` ，查看当前ip地址 `getip` 。

### 浏览器使用代理

见下方的 `Google Chrome` 。  

### KDE桌面代理

`System Settings` > NetWork中的`Settings` > `Proxy` > `Use manually specified proxy configurations` > 'SockS Proxy' 设置为 `127.0.0.1` ，端口 `1080`

### proxychains-ng 自动配置代理

当使用Git的时候，需要设置http、https、git://、ssh等方式的代理，非常麻烦，通过 [proxychains](https://github.com/rofl0r/proxychains-ng) ，我们无需再进行复杂的设置，它可以通过hack的方式自动帮我们代理。  

安装： `yay proxychains-ng`  

配置：编辑 `/etc/proxychains.conf` 文件的最后一行为你的socks5代理，如 `socks5 127.0.0.1 1080` 保存即可  

使用： 在需要代理的软件前加上它就行了，如 `proxychains git clone xxx`

## Google Chrome

### 安装

Chrome有三个版本 `Beta` `Dev` `Stable` ，看自己喜好下载。  

```bash
yay google-chrome
```

### 使用

#### 以代理方式启动

> 用于首次帐号登录，同步书签和扩展插件
>
> 确保代理已启动！

```bash
google-chrome-stable --proxy-server="socks5://127.0.0.1:1080"
```

#### 使用SwitchyOmega插件


首先设置自己的本地代理端口如图：

![so1](https://s1.ax1x.com/2018/12/19/FDy2a4.png)  

然后配置 `auto switch`，注意图中红色部分，将GFWLIST填入：  

```text
https://raw.githubusercontent.com/gfwlist/gfwlist/master/gfwlist.txt
```



![so2](https://s1.ax1x.com/2018/12/19/FDyRIJ.png)  

配置好后，该插件会根据规则自动选择使用代理还是直连。  

如果遇到规则里面没有，且打不开的情况，就手动将该网站加入规则中：

选择 `add condition`    

![so3](https://s1.ax1x.com/2018/12/19/FDyhGR.png)  

![so4](https://s1.ax1x.com/2018/12/19/FDyoM6.png)    



## axel

多线程下载神器，`pacman` 和 `yay` 默认都是单线程下载的，通过 `axel` 实现多线程下载。  

```bash
yay axel
```

配置 `pacman.conf` ：  

```bash
sudo vim /etc/pacman.conf
    # 配置 XferCommand
    XferCommand = /usr/bin/axel -a -n 16 %u -o %o
sudo pacman -Syyu
```

配置 `makepkg.conf` ，让AUR资源使用多线程：

```bash
sudo vim /etc/makepkg.conf
	# 将http::/usr/bin/curl改为下面这行
	'http::/usr/bin/axel -a -n 16 %u -o %o'
yay -Syyu
```

## VLC

视频播放器。  

```
yay -S vlc
```

## cmatrix

大概就是Hacker Hacker，牛叉感十足。  

![cmatrix](https://s1.ax1x.com/2018/12/19/FDybZD.png)  



## Open SSH

使用ssh协议，通过网络和加密通道进行计算器之间的连接通信。  

```bash
yay openssh
```

连接到某服务器：  

```bash
ssh -p <ssh端口> <用户名>@<ip地址>
```

设置常用连接，添加~/.ssh/config：  

```
# host-specific options
Host blogvps
	HostName <ip地址>
	Port     <ssh端口>
	User     <用户名>
```

做如上配置后，可以便捷连接：  

```bash
ssh blogvps
```



## Latte-Dock

OS X那样的应用停靠栏，可定制性高，非常漂亮。  

```bash
yay latte-dock
```


## Team Viewer

远程桌面控制。  

```bash
yay teamviewer
```

## 字体

> 更多Linux字体美化见[Linux下终极字体配置方案](https://ohmyarch.github.io/2017/01/15/Linux%E4%B8%8B%E7%BB%88%E6%9E%81%E5%AD%97%E4%BD%93%E9%85%8D%E7%BD%AE%E6%96%B9%E6%A1%88/)

```bash
# 思源黑体 Noto Sans CJK TC ，TC是T Chinese
yay -S ttf-noto
yay -S adobe-source-han-sans-cn-fonts
# 文泉微米黑
yay -S wqy-microhei
# Android 的字体
yay -S ttf-droid
# Apple 的字体
yay -S ttf-monaco
```

终端我用的Noto Sans Mono for Powerline 或者 Hack。

![font1](https://s1.ax1x.com/2018/12/19/FDyLIH.png)  

chrome我设置的文泉微米黑。  

![font2](https://s1.ax1x.com/2018/12/19/FDyjJA.png)      

编程字体 `monaco`：  

![font4](https://s1.ax1x.com/2018/12/19/FDy4R1.png)  



## Rime 中州韵输入法

Linux端最好用的中文输入法。  

**安装：**  

```bash
yay -S ibus ibus-qt ibus-rime
```

**IBUS配置：**  

在 `~/.zshrc` 中加入：  

```
export GTK_IM_MODULE=ibus
export XMODIFIERS=@im=ibus
export QT_IM_MODULE=ibus
# 保证ibus后台启动
ibus-daemon -x -d
```

执行 `qtconfig-qt4` ，在 "Interface" -> "Default Input Method"中，将默认的输入法设置为"ibus"而不是"xim"。  

然后执行 `ibus-setup` ，添加中文输入法。

**RIME配置：**  

在 `~/.config/ibus/rime` 目录下，创建以下两个文件。  

`default.custom.yaml` 设置默认预选文字为9个  

```yaml
patch:
  "menu/page_size": 9
```

`luna_pinyin.custom.yaml` 设置明月拼音默认为简体输入  

```yaml
patch:
  switches:                   # 注意縮進
    - name: ascii_mode
      reset: 0                # reset 0 的作用是當從其他輸入方案切換到本方案時，
      states: [ 中文, 西文 ]  # 重設爲指定的狀態，而不保留在前一個方案中設定的狀態。
    - name: full_shape        # 選擇輸入方案後通常需要立即輸入中文，故重設 ascii_mode = 0；
      states: [ 半角, 全角 ]  # 而全／半角則可沿用之前方案中的用法。
    - name: simplification
      reset: 1                # 增加這一行：默認啓用「繁→簡」轉換。
      states: [ 漢字, 汉字 ]

```

**使用：**  

点击图标，重新部署，然后通过windows + 空格进行输入法切换。  

## Win10&Linux双系统，系统时间问题

WIN10和Linux系统同时安装后，会发现WIN的时间比实际时间早了8小时。  
在WINDOWS中管理员方式运行 `PowerShell` ,键入以下内容后 `重启` ，即可修复：  

```bash
Reg add HKLM\SYSTEM\CurrentControlSet\Control\TimeZoneInformation /v RealTimeIsUniversal /t REG_DWORD /d 1
```
