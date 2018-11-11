# 搬瓦工VPS建站指南

## KiwiVM 基本配置

服务器购买完毕后，进入 `Client Area -> Services-> My Services` 便能看见服务器列表：  

![clientarea](imgs/clientarea.png)  

查看服务器状态、重装系统、二次验证、修改root密码等操作都是在KiwiVM中进行的。  

### Main controls 系统状态监控

这里能看到系统内存和交换区的占用、IP地址、SSH端口、运行状态、硬盘容量等信息。  

![main](imgs/maincontrols.png)  

### Detailed statistics 状态详情

这里以图表的方式展现网络状态、CPU占用情况、储存I/O状态。  

![detail](imgs/detailstatus.png)  

### 网页版终端操作

`Root shell -basic` 和 `Root shell -advanced` 和 `Root shell -interactive` 都是用来执行指令的，但一般不使用，都是用Openssh或者ssh GUI客户端。  

### Install new OS更换系统

搬瓦工提供了丰富的OS选择，推荐安装 Ubuntu-18.04 ，因为它拥有4.15新内核，可直接开启Google BBR。  

![installos](imgs/installos.png)  

### Two-factor authentication 二次验证

每当进入KiwiVM时，都会要求输入二次验证码，这里支持 `Google身份验证` 和手机短信验证。  

![2auth](imgs/twoauth.png)  

### Root password modification ROOT密码修改

改root密码前，需要在Main Controls中stop服务器。并且只能随机生成密码。  

### KiwiVM password modification KiwiVM登录密码修改

  这里可以随意修改KiwiVM登录密码。

### Migration 数据迁移

通过 `Migrate to another DC` 可以切换主机的地址和IP，前提是IP未被GFW查封。  

![migrate](imgs/migrate.png)  

### Snapshots 建立/导入快照

快照可以对当前的VPS进行全拷贝，可用于遇到突发情况时，快速恢复备份。快照只会存储30天，30天之后自动被删除。当然还有一种名叫胶水快照(Sticky snapshots)的服务，这样的快照不会被自动删除，但每个服务器仅能拥有2个胶水快照。



## Ubuntu系统基础配置

系统安装完毕后，执行 `apt-get update` 和 `apt-get upgrade` 更新以下系统和源。

### 通过SSH连接服务器

> 本地用的Arch Linux，搬瓦工安装的Ubuntu 18

#### 以密码方式连接

在Arch上安装OpenSSH：  

```bash
yay -S openssh
```

连接远程服务器：  

```bash
ssh -p <远程ssh端口> <用户名>@<远程IP地址>
```

输入远程服务器的用户密码即可连接，连接后会保存一个 `SHA256 ECDSA ket fingerprint` ，保存路径是 ` ~/.ssh/known_hosts` 。  

将服务器信息写入 `~/.ssh/config` 配置，更方便地去连接：  

```
Host blogvps
	HostName <远程IP地址>
	Port     <远程SSH端口>
	User     <远程用户名>
```

这样一来，直接输入 `ssh blogvps` 就能连接服务器了。  

#### 以密匙方式连接

 `密匙对` 使用非对称加密，提供了一个更简单、安全的方式去连接服务器，被连接的一方保存公匙，连接的一方保存私匙。  

大概的登录流程是这样的：  

- 远程服务器发现有用户登录时，通过 `公匙` 生成字符串发送给用户
- 用户收到字符串后，通过 `私匙` 对字符串 `加密` ，然后返回给远程服务器
- 远程服务器再次用 `公匙` 对字符串 `解密`，若得到的字符串与之前生成的相同，则认证成功

现在我们要连接远程服务器，那么就要把公匙发过去，自己保留私匙。同理GIT HUB也支持SSH方式连接，GIT HUB保留的也是公匙。  

##### 生成密匙

现在我使用本地Arch Linux生成SSH私匙和公匙，执行 `ssh-keygen` 得到以下输出：  

```bash
Generating public/private rsa key pair.
Enter file in which to save the key (/home/cloudsen/.ssh/id_rsa): 
Enter passphrase (empty for no passphrase): 
Enter same passphrase again: 
Your identification has been saved in /home/cloudsen/.ssh/id_rsa.
Your public key has been saved in /home/cloudsen/.ssh/id_rsa.pub.
The key fingerprint is:
SHA256:dhtaQtxvvBglwo0hgACMIIj2+2xdRfLt4/R3HYpI9Oc cloudsen@GLaDOS
The key's randomart image is:
+---[RSA 2048]----+
|%. .... .        |
|=o.    + * .     |
|.       * B o    |
|   .   . o B .   |
|    .   S B =    |
|   .   . B B * . |
|    o . + + B + o|
|     + .   . E .+|
|                o|
+----[SHA256]-----+


cd .ssh 
ls
>> config  id_rsa  id_rsa.pub  known_hosts
```

`ssh-keygen` 默认是生成 `SHA256` 长度为2048-bit的RSA密匙，平常来说够用了。默认私匙的保存路径是 `/<用户>/.ssh/id_rsa` ，公匙的保存路径是 `/<用户>/.ssh/id_rsa.pub` 。生成的过程中请务必输入 `passphrase密码短语` 获得更安全的保障。  

> 务必确保公匙和私匙的权限是正确的，id_rsa是600，id_rsa.pub是644
>
> 若服务器需要SSH连接其他电脑，生成的.ssh文件夹默认在/root/目录下

##### 将公匙发送给远程服务器

方式有很多种，这里将最方便的一种，使用 `ssh-copy-id` 直接将本地的 `d_rsa.pub` 复制到远程服务器的 `/root/.ssh/authorized_keys` 中。为什么远程服务器上公匙要重命名？不急，下面会提到。执行以下指令：  

```bash
ssh-copy-id -p <远程SSH端口> <用户名>@<远程IP>
```

输出信息如下：  

```bash
/usr/bin/ssh-copy-id: INFO: Source of key(s) to be installed: "/home/cloudsen/.ssh/id_rsa.pub"
/usr/bin/ssh-copy-id: INFO: attempting to log in with the new key(s), to filter out any that are already installed
/usr/bin/ssh-copy-id: INFO: 1 key(s) remain to be installed -- if you are prompted now it is to install the new keys
root@xxx.xxx.xx.xxx's password: 
Number of key(s) added: 1
Now try logging into the machine, with:   "ssh -p 'xxxx' 'root@xxx.xxx.xx.xxx'"
and check to make sure that only the key(s) you wanted were added.
```

然后检查远程服务器中是否收有名叫 `authorized_keys` 权限为 `rw` 的公匙：  

```bash
root@ubuntu:~/.ssh# ls -la
total 20
drwx------ 2 root root 4096 Nov 11 10:04 .
drwx------ 6 root root 4096 Nov 11 10:09 ..
-rw------- 1 root root  397 Nov 11 10:04 authorized_keys
-rw------- 1 root root 1766 Nov 11 09:15 id_rsa
-rw-r--r-- 1 root root  393 Nov 11 09:15 id_rsa.pub
```

##### 使用密匙连接

SSH还有一个配置文件存放在 `/etc/ssh/sshd_config`，打开这个文件，去掉如下配置的注释：  

```
PubkeyAuthentication yes
```

再次连接我们的远程服务器，看到提示信息变为了：  

```bash
ssh blogvps
Enter passphrase for key '/home/cloudsen/.ssh/id_rsa':
```

输入我们私匙的密码，就能连接了，不用再输入搬瓦工自动生成的复杂密码了~！  

等等，为什么远程端的公匙要命名为authorized_keys呢？在 `/etc/ssh/sshd_config`中可以看到：  

```
# Expect .ssh/authorized_keys2 to be disregarded by default in future.
#AuthorizedKeysFile     .ssh/authorized_keys .ssh/authorized_keys2
```

配置指定的文件名就是authorized_keys，它存放了所有认证的公匙信息和用户信息，以后若还要添加公匙的话，就在这个文件里面追加。  

#### 增加SSH的安全





#### SSH相关资料

- [SSH keys](https://wiki.archlinux.org/index.php/SSH_keys_(%E7%AE%80%E4%BD%93%E4%B8%AD%E6%96%87)#.E9.80.89.E6.8B.A9.E5.90.88.E9.80.82.E7.9A.84.E5.8A.A0.E5.AF.86.E6.96.B9.E5.BC.8F)