[TOC]

#  Arch Linux 安装(BIOS和虚拟机)

## 初始化设置

> Arch在启动后，首先会进行初始化设置，分区等操作，之后才是正式的安装。

### 检查主板模式

现在主板都是UEFI的，BIOS安装方式有点区别。若是在虚拟机中安装的，用BIOS方式安装就好了。  

```shell
ls /sys/firmware/efi/efivars
```



### 检查网络连接

Arch正常的安装是需要联网的，这一步至关重要。  

```shell
# 有线网使用dhcpcd获取ip
dhcpcd
# 无线网先连接wifi
wifi-menu
# 若能ping通，则网络没问题
ping www.baidu.com
```

### 检查系统时间

首先查看当前时间和时区：  

```shell
timedatectl status
```

可以看到如下输出：  

![sys_time](https://s1.ax1x.com/2018/12/19/FDyl8I.png)  

目前时区还不对，要修改到上海：  

![time_zone](https://s1.ax1x.com/2018/12/19/FDy12t.png)  

```shell
# 查询所有时区，按q退出
timedatectl list-timezones
# 设置时区为亚洲上海
timedatectl set-timezone Asia/Shanghai
# 设置ntp，确保系统时间正确
timedatectl set-ntp true
```

再次查询时间，可以看到local time已经正确：  

![local_time](https://s1.ax1x.com/2018/12/19/FDyQPA.png)  

### 硬盘分区

目的：创建根目录(`/`)、交换区(swap)、用户区(home)。

> 交换分区若内存够大或的固态硬盘就不需要设置，或者给8G就够了。
>
> EFI安装还需要分一个512M的引导分区。
>
> 根目录给个40G就够了，剩下的都给home。

查看当前所有硬盘，忽略盘符中含`rom`，`loop`或`airoot`的硬盘信息：

```shell
fdisk -l
```

![disk_list](https://s1.ax1x.com/2018/12/19/FDyGKf.png)  

对sda硬盘进行操作：  

```shell
fdisk /dev/sda
```

![fdisk](https://s1.ax1x.com/2018/12/19/FDy0Gn.png)  

现在`fdisk`主要操作说明如下：  

- m：查看帮助
- n：新建分区
- w：保存本次操作结果并退出
- q：不保存操作并退出  

主要操作如下图所示：  

![fdisk1](https://s1.ax1x.com/2018/12/19/FDyDx0.png)  

最后操作完毕后，输入`p`查看分区状况，确认无误后，输入`w`保存并退出：  

![fdisk2](https://s1.ax1x.com/2018/12/19/FDy6qU.png)  

![fdisk3](https://s1.ax1x.com/2018/12/19/FDy3xP.png)  

然后对分好的区进行格式化：  

```shell
# 根目录和home格式化为ext4文件格式
mkfs.ext4 /dev/sda1
mkfs.ext4 /dev/sda3
# 交换区使用mkswap
mkswap /dev/sda2
swapon /dev/sda2
```

![fdisk4](https://s1.ax1x.com/2018/12/19/FDyNVg.png)  

最后挂载分区：

```shell
# 根分区挂载到 /mnt目录
mount /dev/sda1 /mnt
# 其他分区需要先创建目录再挂载【只有EFI模式才需要BOOT这个目录，BIOS和在虚拟机安装只对home设置就完了】
#mkdir /mnt/boot
mkdir /mnt/home
mount /dev/sda3 /mnt/home
```

## 安装

分区完毕后，并挂载后，就可以进行正式安装了。

首先要把镜像的源换成国内的：

```shell
vim /etc/pacman.d/mirrorlist
```

用VIM打开源后，键入`/tsinghua`回车，搜索清华源，找到后，键入`dd`剪切清华那一行。  

键入`gg`回到文首，键入`P`粘贴到最前面位置。键入`:wq`保存并退出。

执行一下命令，开始系统安装：

```shell
pacstrap /mnt base base-devel
```

## 配置系统

### Fstab

执行以下指令，生成fstab文件：

```shell
genfstab -U /mnt >> /mnt/etc/fstab
# 查看刚刚生成的文件
vim /mnt/etc/fstab
```

![fstab](https://s1.ax1x.com/2018/12/19/FDyJr8.png)  

### Chroot

```shell
arch-chroot /mnt
```

### 设置时区

将时区设置为上海：

```shell
ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
hwclock --systohc
```

### 安装一些必要软件

```shell
pacman -S vim dialog wpa_supplicant ntfs-3g networkmanager
```

### 本地化设置

使用vim打开本地设置文件，将`zh_CN.UTF-8 UTF-8` `zh_HK.UTF-8 UTF-8` `zh_TW.UTF-8 UTF-8` `en_US.UTF-8 UTF-8` 这四行前面的注释去掉：

```shell
vim /etc/locale.gen
locale-gen
```

![locale](https://s1.ax1x.com/2018/12/19/FDyYqS.png)  

使用VIM打开`/etc/locale.conf`文件，并在行首加入`LANG=en_US.UTF-8`。

### 设置主机名

在该文件第一行加入你的主机名：

```shell
vim /etc/hostname
```

然后需改hosts文件：

```shell
vim /etc/hosts
```

在文末添加以下内容

```text
127.0.0.1 localhost
::1       localhost
127.0.0.1 GLaDOS.localdomain GLaDOS
```

### 设置ROOT密码

```shell
passwd
```

### 创建普通用户，赋予root

```shell
useradd -m -G wheel -s /bin/bash cloudsen
passwd cloudsen
# 打开sudoers文件后，删除wheel组前面的注释（#）
visudo
```

### 安装系统引导器

`grub`是一个启动引导器，同时支持EFI和BIOS方式的启动。  

若使用的UEFI方式引导系统，则还需要安装`efibootmgr`。  

如果是双系统的话，还需要安装`os-prober`，且如果使用Intel CPU的话，则需要安装 `intel-ucode`。

```shell
# 英特尔CPU需要
pacman -S intel-ucode
# 安装grub
pacman -S grub
grub-install --target=i386-pc /dev/sda
grub-mkconfig -o /boot/grub/grub.cfg
```

![grub](https://s1.ax1x.com/2018/12/19/FDyyrT.png)  

### 退出CHROOT重启系统

```shell
exit
reboot
```

登陆自己的账号后，基本安装就结束了，接下来要安装`KDE`图形界面。  

![login](https://s1.ax1x.com/2018/12/19/FDysMV.png)  

### dhcp服务开机自启动

```bash
systemctl enable dhcpcd
```



## 安装KDE5图形界面

```bash
pacman -S plasma-desktop
pacman -S kdebase
pacman -S ttf-dejavu ttf-liberation wqy-microhei
pacman -S alsa-utils pulseaudio pulseaudio-alsa
pacman -S sddm
systemctl enable sddm
reboot
```



