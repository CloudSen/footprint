[toc]

# 前言

## 预准备

- Linux（本文使用Arch Linux）
- Maven
- 能使用maven打包的Java项目

## 将获得

- Sonatype帐号
- PGP Keys
- 能发布到Maven中央库的开发环境

# 注册Sonatype帐号

去[Sonatype官方网站](https://issues.sonatype.org/secure/Dashboard.jspa)自行注册帐号，然后创建一个组织。  

# 生成PGP KEYS

所有发布到中央库的JAR包都必须提供PGP密钥。因此，你需要生成 PGP 密钥对，然后将公钥发布到服务器，最后将需要发布的JAR包使用PGP签名。私钥需要自己妥善保管，不要对外公布。用户从中央Maven存储库下载JAR文件时，使用公钥来验证签名。PGP密钥对是有失效时长的，当PGP密钥对失效后，仅需要重新生成，然后重新发布公钥即可。  

## 安装GnuPG

> 参考[Arch Wiki](https://wiki.archlinux.org/index.php/GnuPG)

```
yay -S gnupg
```

## 创建密钥对

键入以下指令：  

```
gpg --full-gen-key
```

出现以下提示，直接回车，选择默认即可：  

```
gpg (GnuPG) 2.2.23; Copyright (C) 2020 Free Software Foundation, Inc.
This is free software: you are free to change and redistribute it.
There is NO WARRANTY, to the extent permitted by law.

gpg: 目录‘/home/clouds3n/.gnupg’已创建
gpg: 钥匙箱‘/home/clouds3n/.gnupg/pubring.kbx’已创建
请选择您要使用的密钥类型：
   (1) RSA 和 RSA （默认）
   (2) DSA 和 Elgamal
   (3) DSA（仅用于签名）
   (4) RSA（仅用于签名）
  (14) Existing key from card
```

根据需求选择密钥长度：  

```
RSA 密钥的长度应在 1024 位与 4096 位之间。
您想要使用的密钥长度？(3072)
```

根据需求选择密钥有效期限：  

```
请设定这个密钥的有效期限。
         0 = 密钥永不过期
      <n>  = 密钥在 n 天后过期
      <n>w = 密钥在 n 周后过期
      <n>m = 密钥在 n 月后过期
      <n>y = 密钥在 n 年后过期
密钥的有效期限是？(0)
```

添加用户标识user-id：  

```
GnuPG 需要构建用户标识以辨认您的密钥。

真实姓名： clouds3n
电子邮件地址： www.402645063@gmail.com
注释： 
```

PGP密钥生成完毕：  

```
我们需要生成大量的随机字节。在质数生成期间做些其他操作（敲打键盘
、移动鼠标、读写硬盘之类的）将会是一个不错的主意；这会让随机数
发生器有更好的机会获得足够的熵。
gpg: /home/clouds3n/.gnupg/trustdb.gpg：建立了信任度数据库
gpg: 密钥 xxxxxx 被标记为绝对信任
gpg: 目录‘/home/clouds3n/.gnupg/openpgp-revocs.d’已创建
gpg: 吊销证书已被存储为‘/home/clouds3n/.gnupg/openpgp-revocs.d/xxxxxxAEC27F939E8CECA44BCF58.rev’
公钥和私钥已经生成并被签名。

pub   rsa4096 2020-11-16 [SC]
      xxxxxFD8AEC27F939E8CECA44BCF58
uid                      clouds3n <www.402645063@gmail.com>
sub   rsa4096 2020-11-16 [E]
```

列出公钥环：`gpg --list-keys`  

列出私钥环：`gpg --list-secret-keys`  

导出ASCII版本的公钥： `gpg --export --armor --output public.key <user-id用户名>`

## 发布公钥到密钥服务器



# 结语