# Arch Linux 日常环境配置

> 前提：没有安装 kde-applications 包。

[TOC]  

## KDE文件管理器

安装KDE桌面的文件管理GUI软件：  

```
pacman -S konqueror
```



## KDE截图工具

快捷键`PrtSc`截图：  

```
pacman -S spectacle
```



## 下拉式终端

全局下拉式终端模拟。支持切换主题，默认 `F12` 弹出下拉框。  

```
pacman -S yakuake
```



## 密钥环

用于存储密码、密钥。  

```
pacman -S gnome-keyring
```



## yay AUR安装神器

```
git clone https://aur.archlinux.org/yay.git
cd yay
```

设置GO代理：  

```
export GOPROXY=https://gocenter.io
export GOPROXY="https://goproxy.io"
export GOPROXY="https://athens.azurefd.net/"
```

编译安装：  

```
makepkg -si
```

彩色输出：将 `/etc/pacman.conf` 文件中 `Color` 的注释去掉。  

## Arch Linux CN 源

 Arch Linux 中文社区驱动的非官方软件仓库，包含许多官方仓库未提供的额外的软件包，以及已有软件的 git 版本等变种。  

编辑`/etc/pacman.conf` 文件，在末尾加入[最近地区的镜像地址](https://github.com/archlinuxcn/mirrorlist-repo)，推荐使用华为云：  

```
[archlinuxcn]
Server = https://repo.huaweicloud.com/archlinuxcn/$arch
```

之后安装 `archlinuxcn-keyring `包以导入 GPG key。  

```
pacman -S archlinuxcn-keyring
```

## 代理软件

常用的带有GUI的代理软件推荐两个：

- electron-ssr
- qv2ray

这两个软件都可以在AUR中进行下载，根据代理协议进行选择。

electron-ssr直接安装即可使用：  

```
yay -S electron-ssr
```

  qv2ray的使用方法见[官网](https://qv2ray.net/)，qv2ray需要额外安装v2ray。  

```
pacman -S v2ray
pacman -Syy qv2ray
```

终端可以通过`proxychains-ng`来自动配置代理：  

```
yay proxychains-ng
```

编辑 `/etc/proxychains.conf` 文件的最后一行为你的socks5代理，如 `socks5 127.0.0.1 1080` 保存即可。在需要代理的软件前加上`proxychains`就行了，如 `proxychains git clone xxx`。  

## 字体显示配置

只要配置得当，Linux下的字体显示效果将是桌面系统(Linux/macOS/Windows)中最好的。  

首先安装字体：  

```
yay -S noto-fonts noto-fonts-cjk noto-fonts-emoji ttf-meslo wqy-microhei

```

[下载](https://github.com/ohmyarch/fontconfig-zh-cn)字体配置文件，放入`~/.config/fontconfig`文件夹下。  

然后刷新字体缓存：  

```
fc-cache --force --verbose
fc-cache-32 --force --verbose
```

## Chrome安装并以代理方式启动

安装Chrome：  

```
yay google-chrome
```

以代理方式启动Chrome：  

```
google-chrome-stable --proxy-server="socks5://127.0.0.1:1089"
```

## Open SSH

使用ssh协议，通过网络和加密通道进行计算器之间的连接通信。  

安装：  

```
yay openssh
```

连接到某服务器：  `ssh -p <ssh端口> <用户名>@<ip地址>`  

设置常用连接，添加 `~/.ssh/config`：  

```
# host-specific options
Host <host名>
	HostName <ip地址>
	Port     <ssh端口>
	User     <用户名>
```

做如上配置后，可以通过`ssh <host名>`便捷连接。  

## Dock停靠栏

推荐Latte-Dock，可定制性高，并且非常漂亮。  

```
yay latte-dock
```

## 强大终端ZSH

用zsh就对了：  

```
yay -S zsh
```



## oh-my-zsh 安装和配置

`oh-my-zsh` 是用来管理 `zsh` 配置的框架，`zsh` 的默认配置及其复杂繁琐，而 `oh-my-zsh` 让 `zsh` 配置降到0门槛。而且它完全兼容 `bash`，并且提供了很多使用的插件和美观的主题。  

安装：  

```
proxychains sh -c "$(proxychains curl -fsSL https://raw.github.com/robbyrussell/oh-my-zsh/master/tools/install.sh)"
```

配置：  

- 配置文件：`~/.zshrc`

- 官方自带插件存放在：`~/.oh-my-zsh/plugins/`

- 第三方插件存放在：`~/.oh-my-zsh/custom/plugins/`

- 官方自带主题存放在：`~/.oh-my-zsh/themes/`

- 第三方主题存放在: `~/.oh-my-zsh/custom/themes/`

主题：

推荐安装 [spaceship](https://github.com/denysdovhan/spaceship-prompt) 主题：  

```
git clone https://github.com/denysdovhan/spaceship-prompt.git "$ZSH_CUSTOM/themes/spaceship-prompt"

ln -s "$ZSH_CUSTOM/themes/spaceship-prompt/spaceship.zsh-theme" "$ZSH_CUSTOM/themes/spaceship.zsh-theme"
```

主题安装完成后，修改`.zshrc`配置文件中的`ZSH_THEME="spaceship"`。  

## 中文输入法

推荐Rime中州韵输入法。个人觉得是Linux端最好用的中文输入法。  

安装：  

```
yay -S ibus ibus-qt ibus-rime
```

IBUS配置：  

在 `~/.zshrc` 中加入：  

```
export GTK_IM_MODULE=ibus
export XMODIFIERS=@im=ibus
export QT_IM_MODULE=ibus
# 保证ibus后台启动
ibus-daemon -x -d
```

然后执行 `ibus-setup` ，添加中文输入法。  

RIME配置：  

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

使用：    

点击图标，重新部署，然后通过windows + 空格进行输入法切换。  

## PDF 阅读器

```
yay okula
```

## 连接蓝牙耳机

Arch Linux一般是安装了`bluez`的，若要连接蓝牙耳机还需要安装`pulseaudio-bluetooth`：  

```
yay pulseaudio-bluetooth
```

再需要使用蓝牙时，再开启蓝牙服务：  

```
systemctl start bluetooth
```

启动pulseaudio服务：  

```
pulseaudio -k                   # 确保没有pulseaudio启动
pulseaudio --start              # 启动pulseaudio服务
```

## Axel多线程下载

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
	# 注意http和https的修改
	DLAGENTS=('file::/usr/bin/curl -gqC - -o %o %u'
          'ftp::/usr/bin/curl -gqfC - --ftp-pasv --retry 3 --retry-delay 3 -o %o %u'
          'http::/usr/bin/axel -a -n 16 %u -o %o'
          'https::/usr/bin/axel -a -n 16 %u -o %o'
          'rsync::/usr/bin/rsync --no-motd -z %u %o'
          'scp::/usr/bin/scp -C %u %o')
yay -Syyu
```

