---
layout: post
title: Eureka 4.0.1 配置
categories: [learn]
description: 记一次特殊字符拼接 URL 未做编码导致的异常
keywords: work, bug
---

## Background

最近在学习 `Spring Cloud`，其中有一个很常用的服务注册与发现组件名叫 `Eureka`。由于笔者看的视频教程中使用的 `Eureka` 和 `Spring Boot` 版本都比较老，因此特意用这篇文章来记录一下如何在 `Spring Boot 3.1.2` 下安装和配置 `Eureka 4.0.1`。

我们需要提供相应的 `username` 和 `password` 来进行连接。 由于我们的用户名和对应密码已经存储在 `Vault` 中，因此在这里，我采用的方法为字符串直接拼接的方式来生成完整的 Connection String。

```java
public MongoClient atlasMongoClient(@Value("${atlas.mongo.username}") final String username,
                                    @Value("${atlas.mongo.password}") final String password) {
    String connectionString = String.format(
            "mongodb+srv://%s:%s@your-database-name.mongodb.net", username, password);
    MongoClientSettings settings = MongoClientSettings.builder()
            .applyConnectionString(new ConnectionString(connectionString))
            .build();

    // Create a new client and connect to the server
    try {
        LOG.info("Using Atlas mongo client");
        return MongoClients.create(settings);
    } catch (MongoException e) {
        LOG.error("Error happened when creating Atlas mongo client");
        e.printStackTrace();
    }
    return null;
}

```

然而在提交了修改之后我发现部署失败了，经过查看应用日志找到根本原因，显示的报错信息为：

```
java.lang.IllegalArgumentException: URLDecoder: Illegal hex characters in escape (%) pattern - For input string: "Fl"
```

## Analysis

首先根据报错信息来看，问题出在我们的 URL 上：出现了非法字符 `%`。通过查看密码内容得知，在我们的密码中确实含有一个 `%`。由此则可以解释问题原因：

在 URL 中， 百分号（%）后面应该跟着两个十六进制字符，表示一个 ASCII 字符的编码。而由于我们的密码中包含了 `%`（假如它的内容为 `ABC%FL`），当我们不做任何处理直接将它拼接进 URL 中时，URL decoder 会将 `%FL` 当成一个 ASCII 字符编码去处理，很显然就会报错了。

## Solution

在使用字符串拼接 URL 之前，我们可以使用 Java 提供的 `URLEncoder` 对字符串进行编码，以便将其作为 URI 参数传递，使用方法如下：

```java
try {
    String encodedPassword = URLEncoder.encode(password, "UTF-8");

    String connectionString = String.format(
        "mongodb+srv://%s:%s@your-database-name.mongodb.net", username, encodedPassword);
} catch (UnsupportedEncodingException e) {
    e.printStackTrace();
}
```

由此问题得以解决。

## 附录：完整 pom 文件

### 父工程

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <packaging>pom</packaging>
    <modules>
        <module>microservice_user</module>
        <module>microservice_movie</module>
        <module>eureka_server</module>
    </modules>
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>3.1.2</version>
        <relativePath/> <!-- lookup parent from repository -->
    </parent>
    <groupId>com.example</groupId>
    <artifactId>sm1234_parent</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <name>sm1234_parent</name>
    <description>sm1234_parent</description>
    <properties>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
        <java.version>17</java.version>
    </properties>
    <dependencies>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>


        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <optional>true</optional>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>

        <!-- https://mvnrepository.com/artifact/org.springframework.cloud/spring-cloud-dependencies -->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-dependencies</artifactId>
            <version>2022.0.2</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>

    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
                <configuration>
                    <excludes>
                        <exclude>
                            <groupId>org.projectlombok</groupId>
                            <artifactId>lombok</artifactId>
                        </exclude>
                    </excludes>
                </configuration>
            </plugin>
        </plugins>
    </build>
</project>

```

### 子模块 - Eureka Server

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <parent>
        <groupId>com.example</groupId>
        <artifactId>sm1234_parent</artifactId>
        <version>0.0.1-SNAPSHOT</version>
    </parent>

    <groupId>cn.sm1234</groupId>
    <artifactId>eureka_server</artifactId>

    <properties>
        <maven.compiler.source>17</maven.compiler.source>
        <maven.compiler.target>17</maven.compiler.target>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    </properties>

    <dependencies>
        <!-- https://mvnrepository.com/artifact/org.springframework.cloud/spring-cloud-starter-netflix-eureka-server -->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-eureka-server</artifactId>
            <version>4.0.1</version>
        </dependency>
    </dependencies>

</project>
```

### 子模块 - Eureka Client

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <parent>
        <groupId>com.example</groupId>
        <artifactId>sm1234_parent</artifactId>
        <version>0.0.1-SNAPSHOT</version>
    </parent>

    <groupId>org.example</groupId>
    <artifactId>microservice_movie</artifactId>

    <properties>
        <maven.compiler.source>17</maven.compiler.source>
        <maven.compiler.target>17</maven.compiler.target>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    </properties>

    <dependencies>

        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
            <version>4.0.1</version>
        </dependency>
    </dependencies>

</project>
```
