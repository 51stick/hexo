---
title: springboot自动配置原理
date: 2017-03-25 10:05:18
categories: springboot
tags: springboot
---

了解过springboot的朋友都知道，springboot是建立在spring的基础之上，并且springboot最大的好处就是减少了开发的配置，这也是springboot最大的特点，也正是因为这个特点使得springboot中很多组件可以做到开箱即用。本文将要探索的就是springboot自动配置的原理。
<!-- more -->

### springboot程序入口
当我们就开始一个spingboot程序时，往往看到的是如下的入口程序。

```java
@SpringBootApplication
public class App {
    public static void main(String[] args) {
        ConfigurableApplicationContext context = SpringApplication.run(App.class, args);
        context.close();
    }
}
```

这是`springboot`程序的入口代码，只需使用`@SpringBootApplication`我们就已经开启了springboot的自动配置功能，这也是springboot帮我们默认就配置好了。那么这段代码中，springboot为就已具备了自动配置功能呢？这段代码没有什么特别的，所以关键在于`@SpringBootApplication`注解，所以下面我们在来看看`@SpringBootApplication`的源码。

### @SpringBootApplication

首先，我们来看看该注解的源码：

```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
@SpringBootConfiguration
@EnableAutoConfiguration
@ComponentScan(excludeFilters = @Filter(type = FilterType.CUSTOM, classes = TypeExcludeFilter.class))
public @interface SpringBootApplication {

	Class<?>[] exclude() default {};

	String[] excludeName() default {};

	@AliasFor(annotation = ComponentScan.class, attribute = "basePackages")
	String[] scanBasePackages() default {};

	@AliasFor(annotation = ComponentScan.class, attribute = "basePackageClasses")
	Class<?>[] scanBasePackageClasses() default {};
}
```

在`@SpringBootApplication`注解源码中，我们可以看到一个`@EnableAutoConfiguration`注解，顾名思义叫"启用自动配置"。所以看到这，可以知道，sprinboot的自动配置功能就是依赖于该注解。因此在springboot的入口程序，也可以直接使用`@EnableAutoConfiguration`来启用springboot的自动配置功能。其实在早期的springboot版本中并没有`@SpringBootApplication`注解，因此早期版本必须使用`@EnableAutoConfiguration`注解显示开启springboot的自动配置功能，只是后面的springboot版本中引入了`@SpringBootApplication`注解，在该注解中集成了`@EnableAutoConfiguration`以及其他默认开启的功能，也就是说它帮我们整合了自动配置的功能。

### @EnableAutoConfiguration

下面我们在来看看`@EnableAutoConfiguration`注解的源码。

```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
@AutoConfigurationPackage
@Import(EnableAutoConfigurationImportSelector.class)
public @interface EnableAutoConfiguration {

   String ENABLED_OVERRIDE_PROPERTY = "spring.boot.enableautoconfiguration";

   Class<?>[] exclude() default {};

   String[] excludeName() default {};
}
```

ENABLED_OVERRIDE_PROPERTY = "spring.boot.enableautoconfiguration"：表示通过properties配置项指定是否开启自动配置功能，默认是true开启。

在`@EnableAutoConfiguration`的源码中我们可以看到，这里使用了`@Import`注解，并且导入了类`EnableAutoConfigurationImportSelector.class`，其实springboot自动配置的神奇魔力也在这。

### EnableAutoConfigurationImportSelector

下面我们来看看该类的源码，看看它是如何实现自动配置的。

```java
public class EnableAutoConfigurationImportSelector
		implements DeferredImportSelector, BeanClassLoaderAware, ResourceLoaderAware,
		BeanFactoryAware, EnvironmentAware, Ordered {
  
  ...
    
    @Override
	public String[] selectImports(AnnotationMetadata metadata) {
		if (!isEnabled(metadata)) {
			return NO_IMPORTS;
		}
		try {
			AnnotationAttributes attributes = getAttributes(metadata);
			List<String> configurations = getCandidateConfigurations(metadata,
					attributes);
			configurations = removeDuplicates(configurations);
			Set<String> exclusions = getExclusions(metadata, attributes);
			configurations.removeAll(exclusions);
			configurations = sort(configurations);
			recordWithConditionEvaluationReport(configurations, exclusions);
			return configurations.toArray(new String[configurations.size()]);
		}
		catch (IOException ex) {
			throw new IllegalStateException(ex);
		}
	}

	protected boolean isEnabled(AnnotationMetadata metadata) {
		if (getClass().equals(EnableAutoConfigurationImportSelector.class)) {
			return this.environment.getProperty(
					EnableAutoConfiguration.ENABLED_OVERRIDE_PROPERTY, Boolean.class,
					true);
		}
		return true;
	}
  
  ...  
  
}
```

源码这里我们省略其他方法，只保留两个关键的方法。

1.isEnabled方法：这里会判断`@EnableAutoConfiguration`注解中的`EnableAutoConfiguration.ENABLED_OVERRIDE_PROPERTY`属性，即前面说的`spring.boot.enableautoconfiguration`属性，从这里也看以看出，如果没有配置该属性，默认自动配置功能是开启的。

2.selectImports方法：该方法是来自于`ImportSelector`接口，`EnableAutoConfigurationImportSelector`类实现了`DeferredImportSelector`接口，而`DeferredImportSelector`接口又继承与`ImportSelector`接口。下面我们再来看看`ImportSelector`接口。

```java
public interface ImportSelector {

	/**
	 * Select and return the names of which class(es) should be imported based on
	 * the {@link AnnotationMetadata} of the importing @{@link Configuration} class.
	 */
	String[] selectImports(AnnotationMetadata importingClassMetadata);
}
```

该接口返回的是一个字符串数组，其值是类的全路径名称，也就是说，返回的数组中的类的全路径所指定的类，最后会被实例化成bean，并且被纳入spring容器所管理起来。

在`EnableAutoConfigurationImportSelector`类中实现了`ImportSelector`接口中的selectImports方法，最终将要自动配置的bean的全路径名以字符串数组返回，然后被spring容器管理起来。这里也就达到了自动配置的功能。那么该实现方法是如何找到并返回要自动加载的bean的类的呢？下面我们在来具体看看`selectImports`方法的实现代码。

### SpringFactoriesLoader类

在`selectImports`方法中有这样一句代码：

```java
List<String> configurations = getCandidateConfigurations(metadata, attributes);
```

这里返回的就是需要自动装配的bean的全路径名的字符串数组，然后我们在来看看`getCandidateConfigurations`方法的实现：

```java
protected List<String> getCandidateConfigurations(AnnotationMetadata metadata,
			AnnotationAttributes attributes) {
		List<String> configurations = SpringFactoriesLoader.loadFactoryNames(
				getSpringFactoriesLoaderFactoryClass(), getBeanClassLoader());
		Assert.notEmpty(configurations,
				"No auto configuration classes found in META-INF/spring.factories. If you "
						+ "are using a custom packaging, make sure that file is correct.");
		return configurations;
}
```

从代码以及断言提示中可以看到，这里会通过`SpringFactoriesLoader`类的`loadFactoryNames`方法在`classpath`路径下的`META-INF`文件夹下去找一个叫`spring.factories`的文件。然后把该文件中以`org.springframework.boot.autoconfigure.EnableAutoConfiguration`作为键的配置项的值返回，其实这个配置文件中`org.springframework.boot.autoconfigure.EnableAutoConfiguration`配置项的值，就是需要springboot自动装配的bean的类全路径名称。我们可以看看springboot的自动配置组件`spring-boot-autoconfigure`下的组织结构：

![Markdown](http://i2.buimg.com/1949/1eefc0cfd5b0690f.png)

从图中，我们可以看到，这里配置了很多需要自动配置的bean的类的全路径名称。这些就是springboot在启动时默认会装配的bean。
因此，`SpringFactoriesLoader.loadFactoryNames`方法会查找classpath路径下所有`META-INF/spring.factories`文件中对应的`org.springframework.boot.autoconfigure.EnableAutoConfiguration`配置项的值，并返回。

### 自动配置总结
到这里，springboot自动配置的过程也就是分析完了，下面我们总结一下：
1.`@SpringBootApplication`集成`@EnableAutoConfiguration`。
2.`@EnableAutoConfiguration`导入了`EnableAutoConfigurationImportSelector.class`。
3.`EnableAutoConfigurationImportSelector`最终实现了`ImportSelector`接口的`selectImports`方法。
4.`ImportSelector`接口的`selectImports`中，通过`SpringFactoriesLoader`类的`loadFactoryNames`方法，加载classpath下所有`META-INF/spring.factories`文件中的`org.springframework.boot.autoconfigure.EnableAutoConfiguration`配置的类的全路径名。
5.将`ImportSelector`接口的`selectImports`返回的全路径名对应的类纳入spring容器管理。

到此，springboot的自动配置也就分析完了。
