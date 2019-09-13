---
title: springboot的日志配置
date: 2017-03-19 13:24:28
categories: springboot
tags: springboot
---

本文将要介绍的是springboot中日志组件的相关配置，springboot默认使用的是logback日志框架，同时本文也会简单介绍在springboot如何配置并使用其他的日志框架，log4j2。
<!-- more -->

### springboot默认日志框架是logback

正如文章开头所说，springboot中默认使用的日志组件为`logback`，那么它是怎样引入的logback呢?

在使用springboot时，我们会引入如下的maven配置：

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter</artifactId>
</dependency>
```

在`spring-boot-starter`中又引入了如下的日志配置组件：

```xml
<dependencies>
	......
	<dependency>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter-logging</artifactId>
	</dependency>
	......
</dependencies>
```

而在`spring-boot-starter-logging`中，正式引入了`logback`的配置，如：

```xml
<dependencies>
	<dependency>
		<groupId>ch.qos.logback</groupId>
		<artifactId>logback-classic</artifactId>
	</dependency>
	<dependency>
		<groupId>org.slf4j</groupId>
		<artifactId>jcl-over-slf4j</artifactId>
	</dependency>
	<dependency>
		<groupId>org.slf4j</groupId>
		<artifactId>jul-to-slf4j</artifactId>
	</dependency>
	<dependency>
		<groupId>org.slf4j</groupId>
		<artifactId>log4j-over-slf4j</artifactId>
	</dependency>
</dependencies>
```

然后根据maven的配置依赖关系我们可以知道，springboot在引入启动的基础依赖包中就导入了logbak的组件。

### logbak的配置方式

#### 在application.properties中配置

在springboot中，我们可以直接在`application.properties`配置文件中配置`logback`日志组件的相关属性，这种方式简单粗暴，但是实践中不会这么干。如：

```
logging.level.com.springboot.learn.service=debug

logging.file=e:/tmp/my.log

logging.pattern.console=%-20(%d{yyyy-MM-dd HH:mm:ss.SSS} [%thread]) %-5level %logger{80} - %msg%n

logging.file.console=%-20(%d{yyyy-MM-dd HH:mm:ss.SSS} [%thread]) %-5level %logger{80} - %msg%n
```

`说明：`
1.logging.level.xxx：该配置项指定日志的输出级别。如果xxx=root，表示所有日志的输出级别；如果xxx=package|class，表示指定该package或则class日志级别，日志级别有：`TRACE, DEBUG, INFO, WARN, ERROR, FATAL, OFF`。

2.logging.file:指定日志文件输出的路径和文件名。

3.logging.pattern.console：表示控制台日志文件的输出格式。

4.logging.file.console：表示日志文件输出的日志格式。

#### 在日志配置文件中配置日志

默认情况下，springboot加载classpath下的`logback.xml`或`logback-spring.xml`文件。但是官方推荐使用`logback-spring.xml`命名`logback`的日志配置文件名，下面我们简单看看看如何配置。

在classpath下添加如下`logback-spring.xml`配置文件，内容如下：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<configuration>
    <appender name="consoleLog" class="ch.qos.logback.core.ConsoleAppender">
        <layout class="ch.qos.logback.classic.PatternLayout">
            <pattern>%-20(%d{yyyy-MM-dd HH:mm:ss.SSS} [%thread]) %-5level %logger{80} - %msg%n</pattern>
        </layout>
    </appender>
    <root level="INFO">
        <appender-ref ref="consoleLog"/>
    </root>
</configuration>
```

如上，在`logback-spring.xml`简单的配置了控制台的日志输入格式。

### 使用其他日志组件

如果不想用默认的logback日志组件，我们也可以使用其他的日志组件，配置步骤如下：

#### 排除默认日志依赖组件

从`spring-boot-starter`中排除默认加载的日志组件`spring-boot-starter-logging`，如：

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter</artifactId>
    <exclusions>
        <exclusion>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-logging</artifactId>
        </exclusion>
    </exclusions>
</dependency>
```

#### 引入需要的日志依赖
在排除默认日志依赖后，需要重新加入将要使用的日志依赖，此处加入的`log4j2`的依赖，重新引入依赖后的pom配置如下：

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter</artifactId>
    <exclusions>
        <exclusion>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-logging</artifactId>
        </exclusion>
    </exclusions>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-log4j2</artifactId>
</dependency>
```

#### 在classpath下加入相应的配置文件

在完成了上述两个步骤后，最后我们需要在classpath下加入相应的配置文件。
这里我们加入一个简单的log4j2的日志配置文件`log4j2-spring.xml`,内容如下：

```	xml
<?xml version="1.0" encoding="UTF-8"?>  
<configuration>  
    <appenders>
        <Console name="console" target="SYSTEM_OUT" follow="true">  
            <PatternLayout pattern="%d{yyyy-MM-dd HH:mm:ss.SSS} [%thread] %-5level %logger{80} - %msg%n"/>  
        </Console>
    </appenders>  
    <loggers>   
        <root level="DEBUG">
            <appender-ref ref="console"/>
        </root>
    </loggers>
</configuration>
```

到此，springboot中关于日志的使用部分就介绍完了。