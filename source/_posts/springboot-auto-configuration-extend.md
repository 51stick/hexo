---
title: springboot自动配置扩展
date: 2017-03-26 20:05:34
categories: springboot
tags: springboot
---

上一篇文章介绍了springboot的自动配置原理，大致分析了其内部实现的机制，本文将进一步介绍springboot自动配置中的几个重要类，跟深入的理解其自动配置的实现。
<!-- more -->

### 重点实现类介绍
首先我们要重点几个 与自动配置有关的类：

#### @Import注解
首先让我们来看一下该注解类的源码及其注释：
```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
public @interface Import {

	/**
	 * {@link Configuration}, {@link ImportSelector}, {@link ImportBeanDefinitionRegistrar}
	 * or regular component classes to import.
	 */
	Class<?>[] value();

}
```

该注解只有一个方法，即类类型的数组。从方法的说明可以看出，该注解可以用来导入类，分别如下：
1.导入`配置类`。
2.导入`ImportSelector`接口实现类。
3.导入`ImportBeanDefinitionRegistrar`接口实现类。
4.导入常规的类。

具体什么意思呢？其实就是说，使用该注解导入的类可以自动生成一个bean并纳入spring容器中管理起来，如果导入的类是一个配置类，那么会将该配置类中的所有bean一并都纳入spring容器中管理起来。

### ImportSelector接口
```java
public interface ImportSelector {

	/**
	 * Select and return the names of which class(es) should be imported based on
	 * the {@link AnnotationMetadata} of the importing @{@link Configuration} class.
	 */
	String[] selectImports(AnnotationMetadata importingClassMetadata);
}
```

通过`@Import`注解导入`ImportSelector`接口实现类，将`selectImports`方法返回的类的全路径指定的 bean纳入到spring容器中进行管理起来，详细可以参考`@EnableAutoConfiguration`源码，也可以参考[springboot自动配置原理](http://www.51stick.com/2017/03/25/springboot-auto-configuration-principle/ "springboot自动配置原理")。

### ImportBeanDefinitionRegistrar接口
```java
public interface ImportBeanDefinitionRegistrar {
	/**
	 * Register bean definitions as necessary based on the given annotation metadata of
	 * the importing {@code @Configuration} class.
	 * <p>Note that {@link BeanDefinitionRegistryPostProcessor} types may <em>not</em> be
	 * registered here, due to lifecycle constraints related to {@code @Configuration}
	 * class processing.
	 * @param importingClassMetadata annotation metadata of the importing class
	 * @param registry current bean definition registry
	 */
	public void registerBeanDefinitions(AnnotationMetadata importingClassMetadata, BeanDefinitionRegistry registry);
}
```

`ImportBeanDefinitionRegistrar`接口的作用是说，当使用`@Import`注解导入该类的实现类时，可以从方法`registerBeanDefinitions`的`AnnotationMetadata`参数获取注解的元数据信息，然后通过参数`BeanDefinitionRegistry`向spring容器中动态的注入bean。

`总结：`
在springboot中，自动配置的实现说到底就是通过`@Import`注解与`ImportSelector`接口实现类的组合来实现的。或则是`@Import`注解与`ImportBeanDefinitionRegistrar`接口实现类的组合来实现自动配置的。

### 自己写一个自动配置注解
springboot的自动配置底层实现原理也讲完了，下面自己动手来写一个配置注解。
我们下一个EnableLog注解，该注解带有一个`package`属性，用来指定类的包路径，在指定包路径中的类实例化时，输出实例化信息。

#### 创建@EnableLog注解
```java
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.TYPE)
@Documented
@Import(LogImportBeanDefinitionRegistrar.class)
public @interface EnableLog {
    String[] packages();
}
```
通过使用该注解来指定，输出哪些包中的类实例化时的日志信息。

#### LogImportBeanDefinitionRegistrar
```java
public class LogImportBeanDefinitionRegistrar implements ImportBeanDefinitionRegistrar {

    public void registerBeanDefinitions(AnnotationMetadata importingClassMetadata, BeanDefinitionRegistry registry) {

        Map<String, Object> attr = importingClassMetadata.getAnnotationAttributes(EnableLog.class.getName());
        String[] packArr = (String[]) attr.get("packages");
        List<String> packages = Arrays.asList(packArr);
        System.out.println("packages : " + packages);

        BeanDefinitionBuilder bdb = BeanDefinitionBuilder.rootBeanDefinition(LogBeanPostProcessor.class);
        bdb.addPropertyValue("packages", packages);

        registry.registerBeanDefinition(LogBeanPostProcessor.class.getName(), bdb.getBeanDefinition());

    }
}
```
该类实现了`ImportBeanDefinitionRegistrar`接口，用来获取`@EnableLog`注解的元数据值，并动态的向spring容器中注入`LogBeanPostProcessor`类的实例以及将注解的元数据值设置到动态注入的bean中。

#### LogBeanPostProcessor
```java
public class LogBeanPostProcessor implements BeanPostProcessor {

    // 包名
    private List<String> packages;

    @Override
    public Object postProcessBeforeInitialization(Object bean, String beanName) throws BeansException {
        for (String pack : packages) {
            if (bean.getClass().getName().startsWith(pack)) {
                System.out.println("echo bean : " + bean.getClass().getName());
            }
        }
        return bean;
    }

    @Override
    public Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException {
        return bean;
    }

    public List<String> getPackages() {
        return packages;
    }

    public void setPackages(List<String> packages) {
        this.packages = packages;
    }
}
```
该类实现了`BeanPostProcessor`接口，目的在于bean实例化时，判断bean所在的类的包名是否是`@EnableLog`指定的包名，如果是，那么就输出该类的全路径名。

#### @EnableLog注解测试程序
```java
@SpringBootApplication
@EnableEcho(packages = {"com.springboot.bean", "com.springboot.vo"})
public class App2 {
    public static void main(String[] args) {
        ConfigurableApplicationContext context = SpringApplication.run(App2.class, args);
        context.close();
    }
}
```
上面是`@EnableLog`的测试代码，在springboot启动时，会将指定的两个包下将要实例化的类名的全路径打印出来。

### 总结
最后我们在来总结一下springboot的自动配置的步骤。

#### 方式一
使用`@Import`注解和`ImportSelector`接口实现类完成自动配置，将ImportSelector接口中selectImports方法返回的类实例化并纳入到spring容器中管理起来。

### 方式二
使用`@Import`注解和`ImportBeanDefinitionRegistrar`接口实现类完成自动配置，在ImportBeanDefinitionRegistrar接口的实现类中动态的向spring容器中注入实例并管理起来。

当然实际中的动态配置远比该示例复杂，其中还涉及到了springboot中按条件装配的内容。这些内容与`Condition`接口有关，后面有时间再来专门介绍。