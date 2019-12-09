---
layout: post
title: 使用SpringBoot的推荐项目目录结构| SpringBoot
category: SpringBoot
tag: [SpringBoot]
---

## 使用SpringBoot的推荐项目目录结构

### 一、代码层结构

假设根目录为：com.springboot

- 1.工程启动类(ServerApplication.java)置于com.springboot包下

- 2.实体类(domain)置于com.springboot.domain

    A: com.springboot.domain（jpa项目）
	
    B: com.springboot.pojo（mybatis项目）
	
- 3.数据访问层(Dao)置于com.springboot.repository

    A: com.springboot.repository（jpa项目）
	
    B: com.springboot.mapper（mybatis项目）
	
- 4.数据服务层(Service)置于com,springboot.service,数据服务的实现接口(serviceImpl)至于com.springboot.service.impl

- 5.前端控制器(Controller)置于com.springboot.controller

- 6.工具类(utils)置于com.springboot.util

- 7.常量接口类(constant)置于com.springboot.constant

- 8.配置信息类(config)置于com.springboot.config

- 9.数据传输类(dto)置于com.springboot.dto

> 数据传输对象（Data Transfer Object）用于封装多个实体类（domain）之间的关系,不破坏原有的实体类结构

- 10.视图包装对象（vo）置于com.springboot.vo

> 视图包装对象（View Object）用于封装客户端请求的数据，防止部分数据泄露（如：管理员ID）,保证数据安全,不破坏原有的实体类结构

### 二、资源目录结构

根目录:src/main/resources

- 1.项目配置文件：resources/application.yml 或者 resources/application.properties

- 2.静态资源目录：resources/static/

> 用于存放html、css、js、图片等资源

- 3.视图模板目录：resources/templates/

> 用于存放jsp、thymeleaf等模板文件

- 4.mybatis映射文件：resources/mapper/（mybatis项目）

- 5.mybatis配置文件：resources/mapper/config/（mybatis项目）

### 三.entity、model、domain的不同：

- 1.entity,处于业务层，字段必须和数据库字段一样；

- 2.model前端需要什么我们就给什么，是为页面提供数据和数据校验的； 

- 3.domain很少用，国外很多项目经常用到, 代表一个对象模块

### 四、常用分层整理

- 1、entity层

同类： model层 = entity层 = domain层

作用： 用于存放我们的实体类，与数据库中的属性值基本保持一致。

- 2、mapper层

同类： mapper层 = dao层

作用： 对数据库进行数据持久化操作，他的方法语句是直接针对数据库操作的

- 3、service层

同类： 只有一个 service层

作用： service层 是针对 controller层的 controller，也就是针对我们使用者。service的 impl 是把mapper和service进行整合的文件。

- 4、controller层

同类： controller层 = web 层

作用： 控制器，导入service层，因为service中的方法是我们使用到的，controller通过接收前端传过来的参数进行业务操作，再将处理结果返回到前端。

### 五.参考地址

地址：https://blog.csdn.net/Auntvt/article/details/80381756

