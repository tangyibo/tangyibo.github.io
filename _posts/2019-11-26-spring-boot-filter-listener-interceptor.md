---
layout: post
title: SpringBoot配置监听器、过滤器和拦截器|SpringBoot框架
category: Springboot
tag: [Springboot]
---

这篇文章主要记录了SpringBoot配置监听器、过滤器和拦截器 。
本文分为以下几个部分：
1. 监听器
2. 过滤器
3. 拦截器


## springboot配置监听器、过滤器和拦截器

- 监听器
>listener是servlet规范中定义的一种特殊类。用于监听servletContext、HttpSession和servletRequest等域对象的创建和销毁事件。监听域对象的属性发生修改的事件。用于在事件发生前、发生后做一些必要的处理。其主要可用于以下方面：1、统计在线人数和在线用户2、系统启动时加载初始化信息3、统计网站访问量4、记录用户访问路径。

- 过滤器
> Filter是Servlet技术中最实用的技术，Web开发人员通过Filter技术，对web服务器管理的所有web资源：例如Jsp, Servlet, 静态图片文件或静态 html 文件等进行拦截，从而实现一些特殊的功能。例如实现URL级别的权限访问控制、过滤敏感词汇、压缩响应信息等一些高级功能。它主要用于对用户请求进行预处理，也可以对HttpServletResponse进行后处理。使用Filter的完整流程：Filter对用户请求进行预处理，接着将请求交给Servlet进行处理并生成响应，最后Filter再对服务器响应进行后处理。

- 拦截器：
Interceptor 在AOP（Aspect-Oriented Programming）中用于在某个方法或字段被访问之前，进行拦截然后在之前或之后加入某些操作。比如日志，安全等。一般拦截器方法都是通过动态代理的方式实现。可以通过它来进行权限验证，或者判断用户是否登陆，或者是像12306 判断当前时间是否是购票时间。

## 流程图结构
![structure](https://github.com/tangyibo/tangyibo.github.io/blob/master/_posts/imgs/1090617-20180515204018593-1889287518.png?raw=true)

![structure](https://github.com/tangyibo/tangyibo.github.io/blob/master/_posts/imgs/03c84353cbcfc57dddf7714cba62cfed662.jpg?raw=true)

## 过滤器、拦截器、切片拦截请求的对比

### 相同点： 
- 都可以对请求进行拦截。

### 不同点：
- 过滤器对请求的拦截只能获取到原始的Request 和 Response 的信息。
- 拦截器对请求的拦截可以获取原始的Request、Response和所有的controller及方法名，但无法获取方法的参数信息。
- Aspect切片只能获取方法的参数，原始的Request、Response不能获取。

### 参考地址
- https://www.cnblogs.com/hhhshct/p/8808115.html
- https://www.cnblogs.com/feng9exe/p/11217340.html
