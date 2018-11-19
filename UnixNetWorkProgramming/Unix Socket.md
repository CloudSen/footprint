## 什么是Unix domain socket

Unix domain socket 或者 IPC socket(inter-process communication socket)是在同一个主机中不同进程之间进行数据交换的通信端点。Unix domain socket不仅支持可靠的字节流传输(SOCK_STREAM，和TCP比较)，还支持可靠的数据报传输(SOCK_SEQPACKET)和无序不可靠的数据报传输(SOCK_DGRAM，和UDP比较)。Unix套接字工具是POSIX便携式操作系统的标准组件。  

Unix域套接字的API类似于Internet套接字，而不是使用底层的网络协议，所有通信都完全在操作系统内核中进行。**Unix域套接字使用文件系统作为其地址名称空间**，进程将Unix域套接字引用为文件系统的索引节点(inode)。因此，不同的进程之间可以通过打开相同的socket进行通信。