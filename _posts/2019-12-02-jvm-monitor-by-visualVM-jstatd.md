---
layout: post
title: 利用VisualVM远程监控Java进程|JVM
category: JVM
tag: [JVM]
---

这篇文章主要记录了利用VisualVm和jstatd来远程监控Java进程的方法。
本文分为以下几个部分：
1. 在远程主机上启动jstatd
2. 启动VisualVm


## 利用VisualVM远程监控Java进程

要实现远程监控Java进程，必须在远程主机（运行Java程序的主机）上跑一个jstatd进程，这个进程相当于一个agent，用来收集远程主机上的JVM运行情况，然后用VisualVm连接到这个jstatd，从而实现远程监控的目的。

- 第一步：在远程主机上启动jstatd

要注意的是，jstatd是一个RMI server application，因此在启动时支持java.rmi properties。


根据jstatd文档，我们需要在启动jstatd时提供一个security policy文件：
```
grant codebase "file:${java.home}/../lib/tools.jar" {   
    permission java.security.AllPermission;
};
```

然后运行下面命令启动：
```
jstatd -J-Djava.security.policy=jstatd.all.policy
```

**注意**: 运行jstatd的用户必须和运行Java程序的用户相同，或者是root，否则会监控不到远程主机上的Java进程。

- 第二步：启动VisualVM
远程(Remote)->添加远程主机（Add remote host）->在主机名（Host name）中添加主机的IP地址->确定

就能看到远程主机上的Java进程了。

**注意**如果你点开一个远程进程，那么你会发现有些信息是没有的，比如：CPU、线程、和MBeans。这是正常的，如果需要这些信息（就像监控本地Java进程一样），那么就需要用JMX


### 参考地址
- https://segmentfault.com/a/1190000016634627
