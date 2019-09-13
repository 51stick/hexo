---
title: spring-boot 快速入门
date: 2017-03-11 21:37:08
categories: springboot
tags: springboot
---

本文主要通过一个简单的项目演示`springboot`的基本搭建工作，介绍了`springboot`的基本使用和配置。
<!-- more -->

### 相关介绍
1. 本文使用maven快速构建项目。
2. springboot官方推荐使用gradle来构建springboot项目。
3. [SpringBoot 1.4.2.RELEASE官文档](http://docs.spring.io/spring-boot/docs/1.4.2.RELEASE/reference/htmlsingle/)

### pom依赖配置

当前的pom.xml内容如下，仅引入了两个模块：
```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" 
		 xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 
		 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.springboot.learn</groupId>
    <artifactId>springboot-day01</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <packaging>jar</packaging>

    <name>springboot-day01</name>
    <description>Demo project for Spring Boot</description>

    <properties>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <maven.compiler.source>1.8</maven.compiler.source>
        <maven.compiler.target>1.8</maven.compiler.target>
    </properties>

    <properties>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
        <java.version>1.8</java.version>
    </properties>

    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-dependencies</artifactId>
                <version>1.4.2.RELEASE</version>
                <scope>import</scope>
                <type>pom</type>
            </dependency>
        </dependencies>
    </dependencyManagement>

    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>

</project>
```

`说明`：
1. spring-boot-starter：springboot的核心模块，添加jar包依赖下载springboot的自动配置、日志和YAML等。
2. spring-boot-starter-web：springboot的web模块。
3. spring-boot-starter-test：springboot的测试模块，包括JUnit、Hamcrest、Mockito等。

### 项目基本结构

![项目结构](http://p1.bqimg.com/1949/47e4fbad8450f2e8.png)

`注意：`
1. `src/main/java`符合maven项目的标准结构，这下面是java源代码。
2. `src/main/resources`该目录下存放的是java项目的资源文件。本项目中存放的是springboot的默认配置文件`application.properties`
3. `src/test/java`存放java测试代码。
4. 访问springboot默认启动地址`http://localhost:8089/`，结果如下
5. ![运行结果](http://p1.bqimg.com/1949/42242d4aa29b8d61.png)
http://p1.bqimg.com/1949/42242d4aa29b8d61.png
该界面是springboot的默认访问界面,到此springboot算是正常启动起来了。

### 编写第一个controller服务类
```java
package com.springboot.learn.web;

import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class HelloController {

    @RequestMapping("/hello")
    public String index() {
        return "Hello World";
    }
    
}

```
启动主程序，打开浏览器访问`http://localhost:8089/hello`，可以看到页面输出内容为：Hello World

### 编写单元测试用例
```java
package com.springboot.learn;

import com.springboot.learn.web.HelloController;
import org.junit.Before;
import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.http.MediaType;
import org.springframework.mock.web.MockServletContext;
import org.springframework.test.context.ContextConfiguration;
import org.springframework.test.context.junit4.SpringRunner;
import org.springframework.test.context.web.WebAppConfiguration;
import org.springframework.test.web.servlet.MockMvc;
import org.springframework.test.web.servlet.request.MockMvcRequestBuilders;
import org.springframework.test.web.servlet.setup.MockMvcBuilders;

import static org.hamcrest.CoreMatchers.equalTo;
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.content;
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.status;

@RunWith(SpringRunner.class)
@ContextConfiguration(classes = MockServletContext.class)
//@SpringBootTest
@WebAppConfiguration
public class AppTest {

    private MockMvc mvc;

    @Before
    public void setUp() throws Exception {
        mvc = MockMvcBuilders.standaloneSetup(new HelloController()).build();
    }

    @Test
    public void testHello() throws Exception {
        mvc.perform(MockMvcRequestBuilders.get("/hello").accept(MediaType.APPLICATION_JSON))
           .andExpect(status().isOk())
           .andExpect(content().string(equalTo("Hello World")));
    }

}

```
说明：
1. 使用`MockServletContext`来构建一个空的`WebApplicationContext`，这样我们创建的`HelloController`就可以在`@Before`函数中创建并传递到`MockMvcBuilders.standaloneSetup（）`函数中。

到此，通过maven构建springboot基础项目已经完成。