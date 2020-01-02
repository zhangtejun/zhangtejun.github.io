---
layout: post
title:  "IOC"
date:   2019-10-21 14:34:34
author: zhangtejun
categories: Java
---
##### IOC容器和依赖反转模式
依赖反转：对依赖的控制，把控制权由具体的业务对象手中转交到平台或者框架中，降低对象设计的复杂度和提搞面向对象系统的可测试式性。
即一个对象如何获取它所依赖的对象的引用，对象之间的相互依赖由IoC容器来管理并完成d对象的注入。

在具体的注入方式中有接口注入（type 1）,setter注入（type 2）,构造器注入（type 3）,在spring的IoC设计中，setter和构造器注入是
最主要的方式，setter是最常见的方式。

##### IoC容器的设计
在Spring IoC容器的设计中，最主要的2个容器系列：
* 实现BeanFactory接口的简单容器
* ApplicationContext应用上下文，它作为容器的高级形态存在,它在简单容器的基础上，增加了很多面向框架的特性。

Spring 通过定义BeanDefinition来管理基于spring的应用中的各种对象以及它们之间的依赖关系。BeanDefinition抽象了我们对Bean的定义
，它是让容器起作用的数据类型。

**IOC容器初始化**
一般来说，容器的启动包括BeanDefinition的resource定位，载入，注册3个基本过程。

* Resource定位：完成BeanDefinition的资源定位，它有ResourceLoader通过统一的Resource接口来完成。这个接口对各种形式的BeanDefinition
都使用了统一的接口。比如：文件系统中的bean定义信息可以由FileSystemResource来抽象，在类路径中的Bean定义信息可以由ClassPathResource来实现。
* BeanDefinition 载入： 把用户定义好的bean表示成IOC容器内部的数据结构（BeanDefinition），这个BeanDefinition 是IOC对POJO的抽象
，通过BeanDefinition IOC容器可以方便的对Bean进行管理。
* 注册BeanDefinition：这个过程通过BeanDefinitionRegister接口的实现来完成，这个过程是把解析过程中得到的BeanDefinition像IOC容器注册。
即把BeanDefinition注入到一个HashMap中，IOC容器就是通过这个HashMap来持有这些BeanDefinition的。

这个过程是IOC容器初始化的过程，一般不包括Bean依赖注入的实现，在Spring IOC的设计中，Bean的载入和依赖注入是2个独立的过程。依赖注入一般发生在
应用第一次getBean向容器获取bean的时候。例外的情况是IOC容器有一个预实例化的配置（BeanDefinition的lazy-init属性）。

IOC容器提供了2种实例化java对象的方法，一中是用BeanUtils，它使用了java反射的功能，另一种是CGLIB。

通过反射取得set方法。把对象注入进去

IOC容器中的Bean生命周期
* Bean实例的创建
* 为Bean实例设置属性
* 调用Bean的初始化方法
* 通过IOC容器使用bean
* 容器关闭调用bean的销毁方法


BeanPostProcessor bean后置处理器是一个监听器，它提供Bean在初始之前和之后的回调。
Bean实例化过程`Constructor > @PostConstruct > InitializingBean > init-method`

**autowiring(字段装配)**
Spring的IOC容器还提供自动依赖装配的方式来管理bean。在自动装配中，不需要对Bean属性做显示的依赖声明，IOC会根据这个属性配置，使用反射自动
根据类型或名字来自动匹配bean完成依赖注入。

**bean对IOC容器的感知**
* BeanNameAware 获取bean在IOC容器中的实例名称（ID）
* BeanFactoryAware 在bean中获取bean所在的IOC容器
* ApplicationContextAware 可以获取bean所在的上下文
* MessageSourceAware 可以在bean中获取消息源
* ApplicationEventPublisherAware 可以在bean中获取应用上下文事件发布器
* ResourceLoaderAware 可以得到ResourceLoader,从而加载外部对应的资源
postProcessBeforeInitialization会在InitializeBean的过程中被调用，从而实现对相关Aware接口的注入
`class ApplicationContextAwareProcessor implements BeanPostProcessor`

