---
title: springboot 配置详解
date: 2017-03-12 18:27:57
categories: springboot
tags: springboot
---

本文主要探索springboot中配置文件加载的几种方式，以及有哪些方式可以从springboot配置文件中获取配置项。最后总结了在获取配置文件配置项时，如果有相同配置项，那么这些具有相同的配置项，最终到底使用那种方式加载的配置文件中的内容，即配置文件配置项使用的优先既关系。
<!-- more -->

### 默认配置文件名
springboot中默认的配置文件名如下：

1. application.properties，application-default.properties
2. application.yml，application-default.yml

### 默认配置文件路径

1. classpath 根目录
2. classpath:/config

### 自定义配置文件加载

#### 程序启动参数中指定配置文件

使用程序启动参数加载配置文件有两种方式，一种是加载类路径配置文件，另一种是加载文件系统配置文件

1.在启动参数中使用`--spring.config.name`加载自定义配置文件，如：

```java
--spring.config.name=jdbc
```

`提示：`
	`jdbc`可以是`jdbc.properties`或`jdbc.yml`，

2.在启动参数中使用`--spring.config.location`加载自定义配置文件，如：

```java
--spring.config.location=classpath:jdbc.yml,file:D:/jdbc2.properties
```

![](http://p1.bpimg.com/1949/7b4b77134a5b9518.png)

`提示:`
 	
a) 使用这种方式加载配置文件，必须指定配置文件的全路径名。
b) 这种方式可以指指定多个配置文件，使用`逗号`分隔。

#### 使用spring注解加载配置文件

1.使用`@PropertySource`加载自定义配置文件，如：

```java
@Configuration
@PropertySource("classpath:application.properties")
@PropertySource("file:/e:/tmp/application2.properties")
public class CustomConfigLoad {

}
```

`提示:`

a) 使用`@PropertySource`可以同时加载`类路径(classpath:)`和`系统文件路径（file:）`配置文件。
b) 使用此注解如果要在配置类中加载多个配置文件，只须注解多次即可。

2.使用`@PropertySources`加载自定义配置文件，如：

```java
@Configuration
@PropertySources({
        @PropertySource("classpath:application2.properties"),
        @PropertySource("file:/e:/tmp/application2.properties")
})
public class CustomConfigLoad {

}
```

`提示:`

a) 使用`@PropertySources`和使用`@PropertySource`效果一样。

#### 使用`EnvironmentPostProcessor`实现类加载配置文件

```java
package com.springboo.learn;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.env.EnvironmentPostProcessor;
import org.springframework.core.env.ConfigurableEnvironment;
import org.springframework.core.env.PropertiesPropertySource;
import org.springframework.stereotype.Component;

import java.io.FileInputStream;
import java.io.IOException;
import java.io.InputStream;
import java.util.Properties;

@Component
public class CunstomEnvironmentPostProcessor implements EnvironmentPostProcessor {
    @Override
    public void postProcessEnvironment(ConfigurableEnvironment environment, SpringApplication application) {
        try (InputStream input = new FileInputStream("D:/application.properties")) {
            Properties source = new Properties();
            source.load(input);
            PropertiesPropertySource propertySource = new PropertiesPropertySource("my", source);
            environment.getPropertySources().addLast(propertySource);
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```

`注意:`

a) 使用这种方式必在`META-INF/spring.factories`中以键值对的形式指定该类，源码是这样说的：

```java
import org.springframework.boot.SpringApplication;
import org.springframework.core.env.ConfigurableEnvironment;
import org.springframework.core.env.Environment;

/**
 * Allows for customization of the application's {@link Environment} prior to the
 * application context being refreshed.
 * <p>
 * EnvironmentPostProcessor implementations have to be registered in
 * {@code META-INF/spring.factories}, using the fully qualified name of this class as the
 * key.
 * <p>
 * {@code EnvironmentPostProcessor} processors are encouraged to detect whether Spring's
 * {@link org.springframework.core.Ordered Ordered} interface has been implemented or if
 * the @{@link org.springframework.core.annotation.Order Order} annotation is present and
 * to sort instances accordingly if so prior to invocation.
 *
 * @author Andy Wilkinson
 * @author Stephane Nicoll
 * @since 1.3.0
 */
public interface EnvironmentPostProcessor {

	/**
	 * Post-process the given {@code environment}.
	 * @param environment the environment to post-process
	 * @param application the application to which the environment belongs
	 */
	void postProcessEnvironment(ConfigurableEnvironment environment, SpringApplication application);

}

```

所我们在`META-INF/spring.factories`中作如下配置：

```properties
org.springframework.boot.env.EnvironmentPostProcessor=com.springboo.learn.CunstomEnvironmentPostProcessor
```

这样，就可以在springboot启动的时候，去读取上述示例中的`D:/application.properties`文件。

b) 与`@PropertySource`和`@PropertySources`不同的是，使用此种方式，不仅可以加载文件路径中的配置文件，也可以加载并读取远程配置中心的配置文件。原理从上述示例中可以看出，该方式是在启动的时候，动态的向spring容器中注册配置文件中的配置项。

### 如何获取配置项

假如有一个配置文件叫`application.properties`，其配置内容如下：

```properties
local.port=8080

local.hosts[0]=192.168.1.101
local.hosts[1]=192.168.1.102
local.hosts[2]=192.168.1.103
```

#### 使用注解获取配置项

使用`@Vaule`注解获取spring配置项，这也是项目中使用最多的方式，如：

在类中的获取方式如下：

```java
@Value("${local.port:8080}")
private Integer port;
```

`注意：`
1.该种方式不用写`setter`和`getter`方法，且spring会自动适配配置项的数据类型。
2.上述示例中的`local.port:8080`表示，如果没有获取到配置文件各种的配置(不指定默认配置项会报异常)，则使用默认的8080端口。

#### 使用spring环境变量对象获取配置项

除了上述方式外，在springboot中还可以使用spring核心包中的`Environment`对象来获取配置文件中的未配置项，如：

在类中的获取方式如下：

```java
@Autowired
private Environment env;

String ip = env.getProperty("local.port", Integer.class)
```

`注意:`
这种方式如可以显示的指定配置文件的数据类型，并且，如果配置文件中没有指定的配置项也不会报错，只是获取的配置内容为`null`。

以上两种方式是spring中获取配置文件通过的方法。下面介绍的方式是springboot中获取配置的特性方式。

#### 使用@ConfigurationProperties获取配置项

依然使用上述的`application.properties`文件为例，在类中可以这样获取：

```java
@Component
@ConfigurationProperties(prefix = "local")
public class ConfigurationPropertiesTest {

	private List<String> hosts;

	private Integer port;

	public List<String> getHosts() {
		return hosts;
	}

	public void setHosts(List<String> hosts) {
		this.hosts = hosts;
	}

	public Integer getPort() {
		return port;
	}

	public void setPort(Integer port) {
		this.port = port;
	}

```
上述方法只要配置项前缀一样，就可以在类中简化获取配置的代码。
使用`@ConfigurationProperties`除了作用在类的申明上，也可以直接商用在配置类`bean`的创建方法上，如：

```java
@Configuration
public class MyConfiguration {

    @Bean
    @ConfigurationProperties(prefix = "local")
    public ConfigurationProperties createConfigurationPropertiesTest() {
        return new ConfigurationProperties();
    }

}
```

到此，2中spring通用获取配置项和1中springboot新特性获取配置项的方式就介绍完了。

#### 配置项使用的优先级关系

开始我们介绍了springboot中配置文件的加载方式，有默认加载`classpath`中的`application.properties`等，有自定义加载`classpath`中的其他配置文件，还有有加载`文件系统`和`远程配置`的配置文件。那么此时我们可以思考一个问题，如果这些配置文件中有相同的配置项时，那么最终使用那种配置文件加载方式中的配置项呢？这里我做了一个如下的总结：
1.优先使用`classpath`下默认的`application-default.properties`文件中的配置项(如果指定了profile，该配置文件不会加载)。
2.然后使用`classpath`下默认的`application.properties`文件中的配置项。
3.其次使用`classpath`自定义配置文件中的配置项。
4.最后使用文件系统和远程配置中的自定义配置项。

以上内容就是关于springboot相关配置加载的相干内容，如有总结得不对的地方还请指正。
