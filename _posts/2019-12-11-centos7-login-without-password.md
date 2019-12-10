---
layout: post
title: CentOS7做ssh免密登录| linux
category: linux
tag: [linux]
---

## CentOS7做ssh免密登录

使用ssh-keygen生成公钥和私钥（这里使用默认的rsa），一路默认即可:
```
[root@localhost ~]# ssh-keygen -t rsa　　//默认指定的是rsa，所以可以没有-t rsa
```
在没有指定生成地址时，会默认生成到家目录下的.ssh/目录下。使用rsa就会生成id_rsa和id_rsa.pub两个文件，如果使用的是dsa则生成的是id_dsa和id_dsa.pub两个文件。
```
[root@localhost ~]# ls /root/.ssh/
id_rsa  id_rsa.pub
```

接着使用命令ssh-copy-id命令将公钥发到youxi2服务器上:
```
ssh-copy-id -i id_rsa.pub -p 22 root@192.168.1.3
```

参考地址：https://www.cnblogs.com/diantong/p/10852042.html