---
layout: post
title: Maven入门
categories: Maven
description: Maven简介及使用方法
keywords: Maven, introduction
---

## 简介

Apache Maven是一个软件项目管理工具，主要服务于基于Java平台的项目构建，依赖管理和项目信息管理。 

使用Maven管理项目时，项目依赖的 jar 包将不在项目内，而是集中放置在用户目录下的.m2文件夹下。

## Maven 的 pom.xml

Maven 是基于项目对象模型的概念（POM，Project Object Model）运作的，所以Maven的项目都有一个pom.xml用来管理项目的依赖以及项目的编译功能。

主要需要关注的标签：

* <dependencies>元素

  包含多个项目依赖需要使用的 <dependency>元素

* <dependency>元素

  内部通过 groupId, artifactId 以及 version 确定唯一的依赖

  * groupId: 组织的唯一标识
  * artifactId: 项目的唯一标识
  * version: 项目的版本

* <properities>元素

  变量定义

* <plugins> 和 <plugin>

  编译插件

## Maven的运作方式

Maven会自动根据 dependency 中的依赖配置，直接通过互联网在 Maven 中心库下载相关的依赖包到 .m2 目录下。

## Maven 项目结构

注意：所有目录名字都是固定的，不要自己改大小写

* 根目录
  * src/
    * main/
      * java/
      * resources/
    * test/
      * java/
      * resources/
  * pom.xml

## 启动项目

1. 编码

在 src/main/java 目录下建立包，之后在包中编写Java文件



2. 更改 maven 默认下载路径

maven 会自动将依赖的包先下载到本地，对于Windows 系统来说会默认下载到C盘中，因此最好将默认下载目录更换到其他盘，具体方法如下：

进入maven安装目录中的conf目录，打开settings.xml。大概在53行左右的位置会出现本地仓库地址：

```xml
  <localRepository>/path/to/local/repo</localRepository>
```

 然后修改/pah/to/local/repo目录为自己想要的默认安装目录即可。需要注意的是，Windows中需要用反斜线`\`来代替`/`，需要手动改回去



3. 更改默认下载源

如果是国内用户，推荐使用阿里巴巴下载源。在settings.xml大概150行的位置，由<mirror>标签指出了默认下载源。需要注意的是，添加的下载源应该被包含在<mirrors>标签内。



4. 编译

1. 进入项目的根目录
2. mvn compile命令

