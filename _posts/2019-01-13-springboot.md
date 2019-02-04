---
layout: post
title:  "springboot"
date:   2019-01-13 13:35:21
author: zhangtejun
categories: springboot
---
# springboot

## 启动引导类

`@SpringBootApplication`是一个复合注解,用于开启组件扫描和自动配置。
```java
@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
@SpringBootConfiguration # 该注解和@Configuration作用相同，表示其标注的类是IoC容器的配置类。
@EnableAutoConfiguration # 该注解用于将所有符合自动配置的Bean加载到到前springboot创建并使用的IoC容器中。
@ComponentScan(
    excludeFilters = {@Filter(
    type = FilterType.CUSTOM,
    classes = {TypeExcludeFilter.class}
), @Filter(
    type = FilterType.CUSTOM,
    classes = {AutoConfigurationExcludeFilter.class}
)}
) # 该注解用于自动扫描和加载符合条件的组件或者Bean,并将Bean加载到IoC容器中。
public @interface SpringBootApplication {}
```

>@EnableAutoConfiguration
>>由SpringFactoriesLoader类完成自动配置，SpringFactoriesLoader属于Spring框架私有的一种扩展方案，其主要功能就是从指定的配置文件META-INF/spring.factories加载配置。  
>
>>从classpath中搜寻所有的META-INF/spring.factories配置文件，并将其中org.springframework.boot.autoconfigure.EnableutoConfiguration对应的配置项通过反射（Java Refletion）
>>实例化为对应的标注了@Configuration的JavaConfig形式的IoC容器配置类，然后汇总为一个并加载到IoC容器。
[!]
>>1. loadFactories:接口类获取其实现类的实例，这个方法返回的是对象列表
>>2. loadFactoryNames:加载指定的factoryClass并进行实例化。
>>3. loadSpringFactories：加载指定的factoryClass。
>>4. instantiateFactory：对指定的factoryClass进行实例化。  
>>
>>loadFactories先获取ClassLoader，然后调用loadFactoryNames获取factoryNames，loadFactoryNames调用loadSpringFactories获取当前ClassLoader下的所有META-INF/spring.factories文件中配置信息
>>然后通过Map的getOrDefault(factoryClassName, Collections.emptyList())得到factoryNames。
>> 再调用instantiateFactory方法，通过反射获取到的对应的Factory对象。