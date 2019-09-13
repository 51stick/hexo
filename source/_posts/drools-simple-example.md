---
title: Drools简单示例
date: 2017-03-04 16:55:47
categories: drools
tags: drools
---

本文主要通过一个小的事例程序，介绍drools的基本使用，先通过一个简单的例子来感受一下drools的魅力。
<!-- more -->

### 环境介绍
idea + maven + drools-6.4.0-Final

### 项目结构
如图所示：

![Markdown](http://p1.bqimg.com/1949/a69e3428d5b36c31.png)

### 示例代码

#### pom.xml文件
```xml
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0
         http://maven.apache.org/maven-v4_0_0.xsd">

    <modelVersion>4.0.0</modelVersion>
    <groupId>drools.demo</groupId>
    <artifactId>drools-demo</artifactId>
    <version>1.0-SNAPSHOT</version>

    <packaging>jar</packaging>
    <url>http://maven.apache.org</url>

    <properties>
        <runtime.version>6.4.0.Final</runtime.version>
    </properties>

    <repositories>
        <repository>
            <id>jboss-public-repository-group</id>
            <name>JBoss Public Repository Group</name>
            <url>http://repository.jboss.org/nexus/content/groups/public/</url>
            <releases>
                <enabled>true</enabled>
                <updatePolicy>never</updatePolicy>
            </releases>
            <snapshots>
                <enabled>true</enabled>
                <updatePolicy>daily</updatePolicy>
            </snapshots>
        </repository>
    </repositories>

    <dependencies>
        <dependency>
            <groupId>org.kie</groupId>
            <artifactId>kie-api</artifactId>
            <version>${runtime.version}</version>
        </dependency>
        <dependency>
            <groupId>org.kie</groupId>
            <artifactId>kie-internal</artifactId>
            <version>${runtime.version}</version>
        </dependency>
        <dependency>
            <groupId>org.kie</groupId>
            <artifactId>kie-spring</artifactId>
            <version>${runtime.version}</version>
        </dependency>
        <dependency>
            <groupId>org.kie</groupId>
            <artifactId>kie-ci</artifactId>
            <version>${runtime.version}</version>
        </dependency>
        <dependency>
            <groupId>org.jbpm</groupId>
            <artifactId>jbpm-kie-services</artifactId>
            <version>${runtime.version}</version>
        </dependency>

        <dependency>
            <groupId>org.drools</groupId>
            <artifactId>drools-core</artifactId>
            <version>${runtime.version}</version>
        </dependency>
        <dependency>
            <groupId>org.drools</groupId>
            <artifactId>drools-compiler</artifactId>
            <version>${runtime.version}</version>
        </dependency>
        <dependency>
            <groupId>org.drools</groupId>
            <artifactId>drools-decisiontables</artifactId>
            <version>${runtime.version}</version>
        </dependency>
        <dependency>
            <groupId>org.drools</groupId>
            <artifactId>knowledge-api</artifactId>
            <version>${runtime.version}</version>
        </dependency>

        <dependency>
            <groupId>org.jbpm</groupId>
            <artifactId>jbpm-test</artifactId>
            <version>${runtime.version}</version>
        </dependency>

        <dependency>
            <groupId>ch.qos.logback</groupId>
            <artifactId>logback-classic</artifactId>
            <version>1.0.13</version>
        </dependency>

    </dependencies>

</project>
```

#### kmodule.xml文件
```xml
<?xml version="1.0" encoding="UTF-8"?>
<kmodule xmlns="http://jboss.org/kie/6.0.0/kmodule">

    <kbase name="helloworld" packages="com.drools.examples.helloworld">
        <ksession name="ksession-helloworld"/>
    </kbase>

</kmodule>
```

#### 规则文件helloworldRule.drl
```
package com.drools.examples.helloworld

import com.drools.examples.helloworld.Message;

rule "hello-rule"

    salience 1

    when
        $messageFact : Message($status : status == Message.HELLO, $newMessage : message);
    then
        System.out.println("Begin to chagne message of fact");
        $messageFact.setMessage("New message");
        $messageFact.setStatus(Message.GOODBYE);
end
```

#### Message事实对象(普通javabean)
```java
package com.drools.examples.helloworld;

public class Message {

    public static final int HELLO = 0;
    public static final int GOODBYE = 1;

    private String message;
    private int status;
    private int age;
	
	...... setter ......
    ...... getter ......

}
```

#### 测试类DroolsTest.java
```java
package com.drools.examples.helloworld;

import org.kie.api.KieServices;
import org.kie.api.runtime.KieContainer;
import org.kie.api.runtime.KieSession;

public class DroolsTest {

    public static final void main(String[] args) {
        try {
            KieServices ks = KieServices.Factory.get();
            KieContainer kContainer = ks.getKieClasspathContainer();
            KieSession kSession = kContainer.newKieSession("ksession-helloworld");

            Message message = new Message();
            message.setMessage("Origin message");
            message.setStatus(Message.HELLO);
            message.setAge(20);

            System.out.println("Origin ：message=" + message.getMessage() + ", status=" + message.getStatus());

            kSession.insert(message);
            kSession.fireAllRules();

            System.out.println("New    ：message=" + message.getMessage() + ", status=" + message.getStatus());

        } catch (Throwable t) {
            t.printStackTrace();
        }
    }
}
```

#### 测试结果

![Markdown](http://p1.bpimg.com/1949/435f65ab3b68e827.png)

ok，到此，drools的小例子就成功跑起来了。
