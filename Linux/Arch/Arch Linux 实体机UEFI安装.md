## 制作U盘启动盘

使用 `Rufus` 制作启动盘。硬盘格式为 `GPT`，启动方式为 `UEFI`。

## 检查boot模式

```
# ls /sys/firmware/efi/efivars
```

如果命令执行结束后，没有报错，并且显示了很多目录，则当前为 `UEFI` 模式。否则为 `BIOS` 模式。  

## 联网

### 无线网

配置无线网分为两个步骤：

1. 确保无线网卡驱动正确安装并配置网络接口
2. 用合适的工具管理无线网络连接

#### 检查驱动状态

默认的Arch Linux内核是模块化的，硬件驱动也是通过模块加载的，在启动时会通过`udev`去加载正确的驱动模块。  

检查PCI(e)网卡驱动是否正确加载：  

```
$ lspci -k | more
```

检查USB网卡驱动是否正确加载：

```
$ lsusb -v | more
```

以上命令执行完毕后，会打印一些内核驱动，比如：  

```
06:00.0 Network controller: Intel Corporation WiFi Link 5100
	Subsystem: Intel Corporation WiFi Link 5100 AGN
	Kernel driver in use: iwlwifi
	Kernel modules: iwlwifi
```

从上面的信息可以得知：无线网络对应的内核模块名是iwlwifi。

检查无线网接口是否被创建：  

```
# ip link
```

通常，无线网络接口的命名从字母“W”开始，例如， wlan0或wlp2s0。  

然后开启无线网络接口：  

```
# ip link set <无线网络接口名> up
```

> 这一步，笔记本可能会提示 `RTNETLINK answers: Operation not possible due to RF-kill`。那就是笔记本上的网络功能键的问题。
>
> 执行`rfkill list` 查看锁定状态，如果`Wireless Lan` 的状态是`hard-blocked：yes`，则通过笔记本的功能键开启无线网，若是 `hard-blocked：no  soft-blocked: yes`，则执行`rfkill unblock <对应的数字id>`
>
> 解锁后，重复上一步，开启无线网卡接口。

查看网卡驱动模块加载情况：  

```
# dmesg | grep iwlwifi
```

#### 无线网管理工具iwd

官方推荐的是通过 `iwd` 工具包来管理Wi-Fi。iwd包提供了客户端程序`iwctl`、后台程序`iwd`以及Wi-Fi监听工具`iwmon`。我们通过iwctl来操作iwd。  

##### 交互式方式使用iwctl

通过交互式客户端来连接Wi-Fi：  

```
$ iwctl
```

获取帮助：  

```
[iwd]# help
```

查看可用的网卡列表：  

```
[iwd]# device list
```

扫描网络列表：  

```
[iwd]# station <设备名，如wlan0> scan
```

打印扫描到的网络列表：  

```
[iwd]# station <设备名> get-networks
```

连接网络（会提示输入密码）：  

```
[iwd]# station <设备名> connect <WiFi名>
```

##### 命令行方式使用iwctl

查看可用的网卡列表： 

```
$ iwctl device list
```

扫描网络列表：

```
$ iwctl station <设备名，如wlan0> scan
```

打印扫描到的网络列表：  

```
$ iwctl station <设备名> get-networks
```

连接网络：  

```
$ iwctl --passphrase <密码> station <设备名> connect <WiFi名>
```

### 有线网

todo

检查网络是否正常连接：  

```
$ ping www.baidu.com
```



## 更新系统时钟

同步时间，确保系统时钟准确：  

```
# timedatectl set-ntp true
```

检查服务状态：  

```
# timedatectl status
```



## 硬盘分区

### 分区

因为是UEFI方式安装，因此至少需要以下几个分区：  

- /mnt/boot或/mnt/efi：用于存放EFI信息，至少分260M，多系统建议分个500M
- /mnt：根目录，视情况
- [SWPA]：交换区，至少512M，根据内存大小和是否休眠决定大小
- /mnt/home：用户目录，剩下多少分多少

> 笔记本16G内存，需要休眠，就分20G交换空间，32G内存则为38G交换空间

查看可用的硬盘：  

```
# fdisk -l
```

操作需要分区的硬盘：  

```
# fdisk /dev/the_disk_to_be_partitioned
```

### 格式化分区

格式化UEFI分区：  

```
# mkfs.fat -F32 /dev/boot_partition
```

格式化文件系统分区：  

```
# mkfs.ext4 /dev/root_partition
```

格式化交换分区：  

```
# mkswap /dev/swap_partition
```

### 挂载分区

创建挂载点：  

```
# mkdir /mnt/home
# mkdir /mnt/boot
```

挂载根目录：  

```
# mount /dev/root_partition /mnt
```

挂载用户目录：  

```
# mount /dev/home_partition /mnt/home
```

挂载uefi目录：  

```
# mount /dev/boot_partition /mnt/boot
```

启用交换分区：  

```
# swapon /dev/swap_partition
```

### 查看挂载情况 

```
$ blkid -o list
```





## 安装镜像

优选镜像源：  

```
# reflector --verbose --country China --sort rate --save /etc/pacman.d/mirrorlist
```

开始安装：  

```
# pacstrap /mnt base base-devel linux linux-firmware vim
```

## 配置系统

### Fstab

生成fstab文件：  

```
# genfstab -U /mnt >> /mnt/etc/fstab
```

检查生成的文件：  

```
# vim /mnt/etc/fstab
```

### Chroot

切换到新系统：  

```
# arch-chroot /mnt
```

### 设置时间区域

```
ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
```

生成`/etc/adjtime`文件，硬件时钟设置为UTC：  

```
# hwclock --systohc
```

### 本地化

编辑 `/etc/locale.gen`，取消`en_US.UTF-8 UTF-8` 以及中文UTF-8的注释。

然后生成本地文件：  

```
# locale-gen
```

### 网络配置

#### 配置hostname

编辑`/etc/hostname`文件，存入期望的hostname。

#### 配置hosts

编辑`/etc/hosts`文件，存入以下内容：  

```
127.0.0.1	localhost
::1		localhost
127.0.1.1	myhostname.localdomain	myhostname
```

### 配置用户

#### 设置root密码

```
# passwd
```

#### 添加用户

```
# useradd -m -G wheel -s /bin/bash <用户名>
# passwd cloudsen
```

打开sudoers文件，删除wheel组前面的注释（#）：  

```
# visudo
```

### 更新CPU [microcode](https://wiki.archlinux.org/title/Microcode)

amd：

```
# pacman -S amd-ucode
```

intel:

```
# pacman -S intel-ucode
```

后面生成GRUB的时候，会自动检测microcode。

### 安装引导器

```
# pacman -S grub efibootmgr
```

GRUB是Bootloader，而GRUB安装脚本使用EFIBOOTMGR将引导条目写入NVRAM。

> 若是双系统，还需要安装os-prober

> 确保已挂载/boot分区

开始生成GRUB文件：  

```
# grub-install --target=x86_64-efi --efi-directory=/boot --bootloader-id=GRUB --recheck
```

```
# grub-mkconfig -o /boot/grub/grub.cfg
```



## 软件安装

必备软件：

```
# pacman -S git dialog wpa_supplicant ntfs-3g networkmanager network-manager-applet screenfetch
```

安装KDE相关：  

```
# pacman -S xorg plasma sddm
```

重启系统，配置图形桌面：  

```
# systemctl start NetworkManager
# systemctl enable NetworkManager
# systemctl enable sddm
# systemctl start sddm
```

