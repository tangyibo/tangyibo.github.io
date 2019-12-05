---
layout: post
title: eclipse 安装lombok 插件的方法记录| eclipse
category: eclipse
tag: [eclipse]
---

这篇文章主要记录了利用eclipse 安装lombok 插件的方法。
本文分为以下几个部分：
1. 安装插件
2. 引入依赖
3. 使用注解

## eclipse 安装lombok 插件的方法

### 1.安装插件
下载 lombok.jar
> 地址：https://projectlombok.org/download.html
下载后放到eclipse 的安装目录，在eclipse.ini 文件中加入
```
-Xbootclasspath/a:lombok.jar
-javaagent:lombok.jar
```

重启eclipse 生效，如果仍然不行的话，cmd 进入安装目录输入命令：eclipse clean

***注意***：javaagent:lombok.jar 中的lombok.jar 要和jar包名字一致

### 2.引入依赖

```
		<dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <version>1.16.10</version>
        </dependency>
```

或者在SpringBoot项目中可添加如下内容：

```
		<dependency>
			<groupId>org.projectlombok</groupId>
			<artifactId>lombok</artifactId>
			<scope>provided</scope>
		</dependency>
```

### 3.使用注解

- @Setter
- @Getter
- @Data
- @Log(这是一个泛型注解，具体有很多种形式)
- @AllArgsConstructor
- @NoArgsConstructor
- @EqualsAndHashCode
- @NonNull
- @Cleanup
- @ToString
- @RequiredArgsConstructor
- @Value
- @SneakyThrows
- @Synchronized

### 4.参考地址

地址：https://www.jianshu.com/p/6825d8116006
