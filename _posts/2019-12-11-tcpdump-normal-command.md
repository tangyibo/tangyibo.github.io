---
layout: post
title: tcpdump过滤HTTP请求常用命令:| tcp
category: tcp
tag: [tcp]
---

这篇文章主要总结了tcpdump过滤HTTP请求的常用命令。
本文分为以下几个部分：
1. tcpdump过滤HTTP的GET请求
2. tcpdump过滤HTTP的POST请求
3. tcpdump过滤HTTP的请求和响应头信息，以及请求和响应消息体信息


## tcpdump过滤HTTP的GET请求:

```
tcpdump -s 0 -A 'tcp[((tcp[12:1] & 0xf0) >> 2):4] = 0x47455420'
```

## tcpdump过滤HTTP的POST请求:

```
tcpdump -s 0 -A 'tcp dst port 80 and (tcp[((tcp[12:1] & 0xf0) >> 2):4] = 0x504f5354)'
```

## tcpdump过滤HTTP的请求和响应头信息，以及请求和响应消息体信息：

```
tcpdump -A -s 0 'tcp port 80 and (((ip[2:2] - ((ip[0]&0xf)<<2)) - ((tcp[12]&0xf0)>>2)) != 0)'
tcpdump -X -s 0 'tcp port 80 and (((ip[2:2] - ((ip[0]&0xf)<<2)) - ((tcp[12]&0xf0)>>2)) != 0)'
```


参考地址：http://www.nginx.cn/4692.html