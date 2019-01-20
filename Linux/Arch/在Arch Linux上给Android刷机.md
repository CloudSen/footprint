[TOC]

# 在Arch Linux上给Android刷机

> 如果你使用的是Window，不用害怕，一样可以看看，刷机的方法和原理都相同。
>

## 下载必备工具

### Android工具包：

```bash
# android-tools包含了adb和fastboot
yay -S android-tools
yay -S android-udev
```

### TWRP Recovery

[TWRP官网](https://twrp.me/Devices/) 找到自己的设备下载。文件后缀为 `img` 。 

### 要刷入的ROM包

去[XDA](https://forum.xda-developers.com/)上找个自己喜欢的下载。文件后缀为 `zip` 。

### OpenGapps

去[官网](https://opengapps.org/)选择自己手机cpu架构、上一步Android系统的版本、Gapps版本(建议pico版)。文件后缀为 `zip` 。  

### Magisk

开源root工具，拒绝SuperSU从每个人做起。去[这里](https://forum.xda-developers.com/apps/magisk/official-magisk-v7-universal-systemless-t3473445)下载。文件后缀为 `zip` 。  



## 刷机过程

### Arch Linux的前置准备

将当前用户加入 `adbusers` 用户组：  

```bash
gpasswd -a <用户名> adbusers
```

开启手机的USB调试，并连接电脑，然后找到你手机的 `vendor id` 和 `product id` ：  

```bash
lsusb
Bus 001 Device 011: ID 2a70:4ee7 我的手机
```

如上示例，`vendor id = 2a70` `product id = 4ee7` 。  

添加udev规则到 `/etc/udev/rules.d/51-android.rules` ，手动替换idVendor和idProduct两个值：  

```
SUBSYSTEM=="usb", ATTR{idVendor}=="[VENDOR ID]", MODE="0666", GROUP="adbusers"
SUBSYSTEM=="usb",ATTR{idVendor}=="[VENDOR ID]",ATTR{idProduct}=="[PRODUCT ID]",SYMLINK+="android_adb"
SUBSYSTEM=="usb",ATTR{idVendor}=="[VENDOR ID]",ATTR{idProduct}=="[PRODUCT ID]",SYMLINK+="android_fastboot"
```

重新加载规则：  

```bash
sudo udevadm control --reload-rules
```

检测设备：  

```bash
adb devices

List of devices attached
da948755        no permissions; see [http://developer.android.com/tools/device.html]
```

可以看到出现了 `no permissions` ，这时以root权限重启adb服务：  

```bash
sudo adb kill-server
sudo adb start-server
```

再次检测设备：  

```
adb devices

List of devices attached
da948755        unauthorized
```

将手机重启到 `fast boot` 线刷模式，然后以root检测设备：  

```bash
sudo adb reboot bootloader
sudo flashboot devices

da948755        fastboot
```

### 刷入Recovery

将TWRP刷入手机：  

```bash
sudo fastboot flash recovery twrp-3.2.3-0-oneplus3.img 

target reported max download size of 435159040 bytes
Sending 'recovery' (22588 KB)...
OKAY [  0.651s]
Writing 'recovery'...
OKAY [  0.167s]
Finished. Total time: 0.899s
```

然后将手机手动重启到recovery。然后就能看见TWRP界面了。然后检测设备：  

```bash
adb devices

List of devices attached
da948755        recovery
```

### 三清设备

在图形界面进入 `清理 => 高级清除` ，三清设备，我有强迫症，把内部储存也清理了。勾选以下选项，并确认清除：  

- Dalvik / ART Cache
- Cache
- Data
- Internal Storage(可选)

### 刷入各种包

通过ADB向手机发送刷机包：

```bash
cd ~/Downloads
ls
Magisk-v17.1.zip  SkyDragon-OS-P-oneplus3-20181111.zip
open_gapps-arm64-9.0-pico-20181114.zip   twrp-3.2.3-0-oneplus3.img

# 发送刷机包到手机sdcard目录
adb push SkyDragon-OS-P-oneplus3-20181111.zip /sdcard
adb push open_gapps-arm64-9.0-pico-20181114.zip /sdcard
adb push Magisk-v17.1.zip /sdcard
```

在手机上进入 `安装` ，按照以下顺序依次刷入刷机包：  

1. rom刷机包；
2. OpenGapps；
3. Magisk。

刷完之后，重启进入系统。  

# Good Job

aha so nice man :D  

~~ enjoy your new super fucking fantastic Android system~~





