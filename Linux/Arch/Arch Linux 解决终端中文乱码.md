[TOC]

# Arch Linux 解决终端文件中文名乱码

有时，运行 `ls` 指令，会发现有些以中文命名的文件出现乱码，这是因为在Windows上的中文默认编码为 `GBK` ，而Linux编码是 `UTF-8` ，因此出现了乱码。  

解决的方式很简单，就是将 `GBK` 编码的文件名，转换为 `UTF-8` 编码格式即可，这里推荐使用 `convmv` 工具。  

## 安装

```
# pacman -S convmv
```

## 常用说明

```
USAGE: convmv [options] FILE(S)
-f enc     转换前的格式
-t enc     要转换的最终格式
-r         在目录中递归转换
-i         交互模式，确认每一个转换
--nfc      目标文件将是UTF-8的标准格式 (Linux等系统)
--nfd      目标文件将是UTF-8的标准格式 (OS X 等系统.)
--list     列出所有可用的编码格式
--lowmem   keep memory footprint low (see man page)
--map m    apply an additional character mapping
--nosmart  忽略转换已经是UTF-8的文件
--notest   确认给文件重命名，不加它只会列出哪些需要转换
--unescap  把%20转义为空格
```

## 使用

将某个目录下的所有 `GBK` 编码的文件名转换为 `UTF-8`：  

```
convmv -f GBK -t UTF-8 -r --nosmart --notest <目录>
```

将某个目录下的（不包含子目录）所有 `GBK` 编码的文件名转换为 `UTF-8`：  

```
convmv -f GBK -t UTF-8 --nosmart --notest <目录>
```

## 效果

使用前：  

![](https://s2.ax1x.com/2019/10/09/uIwjsg.png)  

使用后：  

![]()

