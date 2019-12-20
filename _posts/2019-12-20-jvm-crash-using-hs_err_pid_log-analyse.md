---
layout: post
title: JVM致命错误日志(hs_err_pid.log)分析:| JVM
category: JVM
tag: [JVM]
---

这篇文章主要总结了JVM致命错误日志(hs_err_pid.log)分析。
本文分为以下几个部分：
1. 日志头文件
2. 导致crash的线程信息
3. 所有线程信息
4. 安全点和锁信息
5. 堆信息
6. 本地代码缓存
7. 编译事件
8. gc相关记录
9. jvm内存映射
10. jvm启动参数
11. 服务器信息


当jvm出现致命错误时，会生成一个错误文件 hs_err_pid<pid>.log，其中包括了导致jvm crash的重要信息，可以通过分析该文件定位到导致crash的根源，从而改善以保证系统稳定。当出现crash时，该文件默认会生成到工作目录下。包含如下几类关键信息：

- 日志头文件

> 日志头文件包含概要信息，简述了导致crash的原因。而导致crash的原因很多，常见的原因有jvm自身的bug，应用程序错误，jvm参数配置不当，服务器资源不足，jni调用错误等。

- 导致crash的线程信息

- 所有线程信息

- 安全点和锁信息

- 堆信息

- 本地代码缓存

- 编译事件

- gc相关记录

- jvm内存映射

- jvm启动参数

- 服务器信息

