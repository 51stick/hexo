---
title: springboot中profiles配置及使用
date: 2017-03-17 22:05:18
categories: springboot
tags: springboot
---

在实际工作中，往往会面临一个问题，那就是多环境配置问题，分别是开发环境，测试环境，生产环境。本文要介绍的就是springboot中的profiles的使用，指定不同环境的配置等实践问题。
<!-- more -->

### profiles的配置方式

在springboot启动的时候，有两种指定profiles的方式，分别是启动参数指定和编程式指定，下面来详细介绍一下。

首先准备本文中需要使用到的配置文件。

１.application.properties配置文件

```properties
jdbc.username = user name
jdbc.password = password
```

2.application-default.properties配置文件

```properties
jdbc.username = defaultuser name
jdbc.password = default password
```

3.application-dev.properties配置文件
```properties
jdbc.username = dev user name
jdbc.password = dev password
```

4.application-test.properties配置文件
```properties
jdbc.username = test user name
```

#### 正常情况下配置项的使用情况

springboot启动入口类：

```java
@SpringBootApplication
public class App {
    public static void main(String[] args) {
    	
    	ConfigurableApplicationContext context = SpringApplication.run(App.class, args);
    	
    	System.out.println(context.getEnvironment().getProperty("jdbc.username"));
    	System.out.println(context.getEnvironment().getProperty("jdbc.password"));

    	context.close();
    }
}
```

启用springboot程序，运行结果如下：

```
default user name
default password
```

由此我们可以看出，正常情况下，在配置项使用的优先级上`application-default.properties`比`application.properties`高。

下面我们来看看如何在spingboot中使用profiles来控制配置文件的使用。

### springbot中profiles的启用方式

#### 在启动参数中配置profiles

在启动参数中配置`--spring.profiles.active`参数项指定profiles，如图：

![](http://p1.bqimg.com/1949/552dcb53f1260a26.png)

在运行上述主程序：

```java
@SpringBootApplication
public class App {
    public static void main(String[] args) {
    	
    	ConfigurableApplicationContext context = SpringApplication.run(App.class, args);
    	
    	System.out.println(context.getEnvironment().getProperty("jdbc.username"));
    	System.out.println(context.getEnvironment().getProperty("jdbc.password"));

    	context.close();
    }
}
```

结果如下：

```
test user name
password
```

通过结果，我们发现，username使用的是`application-test.properties`中的配置项，而password由于在`application-test.properties`中由于没有配置，则使用的是`application.properties`中的配置项。那么需要注意的问题来了，文章开头不是说`application-default.properties`比`application.properties`配置项的优先级高么？为什么这里用的是后者的配置项，而不是前者`application-default.properties`的配置项？其实，原因是，在指定了`profiles`的情况下，`application-default.properties`文件不会被springboot加载，因此，这里使用的是`application.properties`中的password的配置。所以这是我们需要注意的一个小细节的地方。

#### 在程序中配置profiles

稍微改动一点启动的主程序代码，然后在主程序中配置profiles，如下：

```java
@SpringBootApplication
public class App {
    public static void main(String[] args) {
    	
    	SpringApplication app = new SpringApplication(App.class);

		// 指定profiles
    	app.setAdditionalProfiles("dev");

    	ConfigurableApplicationContext context = app.run(args);
		
    	System.out.println(context.getEnvironment().getProperty("jdbc.username"));
    	System.out.println(context.getEnvironment().getProperty("jdbc.password"));

    	context.close();
    }
}
```

结果如下：

```
dev user name
dev password
```

从运行结果我们可以看到，这里使用的是`application-dev.properties`配置文件中的配置项。

`总结：`
1.在没有设置`profiles`时，`application-default.properties`和`application.properties`都会被springboot加载，且`application-default.properties`配置的优先级高于`application.properties`。
2.如果设置了`profiles`，会首先在`profiles`指定的配置文件中查找需要的配置项。但是由于在这种情况下，`application-default.properties`不会被springboot加载，如果有没有找到的相关配置项，那么之后会去`application.properties`配置文件中继续去查找相关的配置项并启动该配置项。

### 使用@Profile注解

上面讲了`profiles`指定配置文件使用的情况，在程序中，我们也可以使用`@Profile`注解来根据`profiles`启动相关的bean。如：

有如下配置类：

```java
@Configuration
public class MyConfiguration {

	@Bean
	public Runnable createRunnable(){
		System.out.println("===== normal =====");
		return () -> {};
	}

	@Bean
	@Profile("test")
	public Runnable createRunnableTest(){
		System.out.println("===== test =====");
		return () -> {};
	}

	@Bean
	@Profile("dev")
	public Runnable createRunnableDev(){
		System.out.println("===== dev =====");
		return () -> {};
	}
}
```

主程序如下：

```java
@SpringBootApplication
public class App {
    public static void main(String[] args) {

    	SpringApplication app = new SpringApplication(App.class);

		//编程的方式指定生效的profile
    	app.setAdditionalProfiles("dev");

    	ConfigurableApplicationContext context = app.run(args);

		context.getBean(MyConfiguration.class);

		context.close();
    }
}

```

启动程序，运行结果如下：

```
===== normal =====
===== dev =====
```

1.从结果我们可以看到，`@Profile("test")`指定的bean没有被spring容器初始化。也就是说，在配置类中，没有指定`@Profile`注解的创建bean的方法默认也会被spring容器初始化，同时对应的`profiles`的`@Profile("dev")`也会被实例化并纳入spring容器进行管理，而其他与指定`profiles`不匹配的`@Profile`的bean则不会被实例化。

2.其实`@Profile`也可以注解在配置类上，表示只有与`profiles`吻合时，`@Profile`注解的配置类中的`@Bean`才会被spring初始化并纳入spring容器中管理起来。