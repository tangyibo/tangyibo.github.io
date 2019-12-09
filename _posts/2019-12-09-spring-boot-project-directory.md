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

- 1.项目配置文件：resources/application.yml

- 2.静态资源目录：resources/static/

> 用于存放html、css、js、图片等资源

- 3.视图模板目录：resources/templates/

> 用于存放jsp、thymeleaf等模板文件

- 4.mybatis映射文件：resources/mapper/（mybatis项目）

- 5.mybatis配置文件：resources/mapper/config/（mybatis项目）

### 4.参考地址

地址：https://blog.csdn.net/Auntvt/article/details/80381756
