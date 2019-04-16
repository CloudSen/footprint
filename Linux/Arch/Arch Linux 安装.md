[TOC]

#  Arch Linux 安装(BIOS和虚拟机)

## 初始化设置

> Arch在启动后，首先会进行初始化设置，分区等操作，之后才是正式的安装。

### 检查主板模式

现在主板都是UEFI的，BIOS安装方式有点区别。若是在虚拟机中安装的，用BIOS方式安装就好了。  

```shell
ls /sys/firmware/efi/efivars
```
如果出现 `No such file or directory` 则说明是 `BIOS` 方式启动的。  


### 检查网络连接

Arch正常的安装是需要联网的，这一步至关重要。 
`dhcpcd` 默认是开启探测**有线**网络设备的。   

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

> 若内存够大或的固态硬盘就不需要设置交换分区，或者给8G就够了。
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

用VIM打开源后，在最上面加上阿里云共享镜像服务站 `Server = http://mirrors.aliyun.com/archlinux/$repo/os/$arch`。然后键入`:wq`保存并退出。

执行以下命令，开始系统安装：

```shell
# base-devel是额外包，它包含了gzip gcc sudo等工具
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
pacman -S xorg
pacman -S plasma kde-applications
pacman -S sddm
systemctl enable sddm
sudo systemctl disable netctl
sudo systemctl enable NetworkManager
sudo pacman -S network-manager-applet
reboot
```

安装xorg时会提示安装显卡驱动，虚拟机千万不要安装nvidia或amd独显的驱动，不然重启会黑屏，只能看见鼠标。  
若发生以上情况，则卸载nvidia驱动即可:  

```bash
# 查看驱动包的名字
pacman -Q|grep nvidia
# 卸载上面显示的驱动包
pacman -R xxxx
reboot
```



## 安装VMtools

``` bash
pacman -S gtkmm3 open-vm-tools
cat /proc/version > /etc/arch-release
systemctl  enable vmtoolsd
systemctl  start vmtoolsd 
systemctl  enable vmware-vmblock-fuse.service
systemctl  start vmware-vmblock-fuse.service
```

<<<<<<< HEAD


##  解决VMware无法调整分辨率

首先使用 `xrandr` 命令列出当前系统可用的分辨率：  

``` bash
Screen 0: minimum 320 x 200, current 800 x 600, maximum 16384 x 16384
Virtual-1 connected primary 800x600+0+0 (normal left inverted right x axis y axis) 0mm x 0mm
   preferred     60.00*+
   2560x1600     59.99  
   1920x1440     60.00  
   1856x1392     60.00  
   1792x1344     60.00  
   1920x1200     59.88  
   1600x1200     60.00  
   1680x1050     59.95  
   1400x1050     59.98  
   1280x1024     60.02  
   1440x900      59.89  
   1280x960      60.00  
   1360x768      60.02  
   1280x800      59.81  
   1152x864      75.00  
   1280x768      59.87  
   1024x768      60.00  
   800x600       60.32  
   640x480       59.94  
Virtual-2 disconnected (normal left inverted right x axis y axis)
Virtual-3 disconnected (normal left inverted right x axis y axis)
Virtual-4 disconnected (normal left inverted right x axis y axis)
Virtual-5 disconnected (normal left inverted right x axis y axis)
Virtual-6 disconnected (normal left inverted right x axis y axis)
Virtual-7 disconnected (normal left inverted right x axis y axis)
Virtual-8 disconnected
```

如果显示器分辨率没有出现在上面的列表中，如2k分辨率(2560*1440)，则需要手动添加。  

首先使用cvt命令，得到某个分辨率的有效扫描频率：  

``` bash
cvt 2560 1440
# 控制台输出如下
# 2560x1440 59.96 Hz (CVT 3.69M9) hsync: 89.52 kHz; pclk: 312.25 MHz
Modeline "2560x1440_60.00"  312.25  2560 2752 3024 3488  1440 1443 1448 1493 -hsync +vsync
```

然后根据以上输出，新建xrandr模式：  

``` bash
xrandr --newmode "2560x1440_60.00"  312.25  2560 2752 3024 3488  1440 1443 1448 1493 -hsync +vsync
```

然后把新增的模式添加到当前的输出设备：  

``` bash
xrandr --addmode Virtual-1 2560x1440_60.00
```

最后设置当前输出设备的分辨率：  

``` bash
xrandr --output Virtual-1 --mode 2560x1440_60.00
```

<span style="color:red;">注意，以上的操作只在本次会话中有效！</span>

若想永久生效，则需要修改 `xorg.conf`，具体操作如下：  



=======
## VMware分辨率自适应

``` bash
pacman -S xf86-input-vmmouse
pacman -S xf86-video-vmware
```
>>>>>>> a59c0c742fb8e72a122840730216a2d64931de6f
