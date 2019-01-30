---
title: 第1章Spring之旅
date: 2016/11/20 20:46:25
updated: 2016/11/20 20:46:25
categories:
- Spring实战第4版读书笔记
tags:
- Spring
---
# 一、简化java开发
本节只是简单介绍相关内容，具体展开会在后面娓娓道来。
## (1)简述如何简化java开发
4种关键策略：
1. 基于POJO(简单的Java对象，普通JavaBeans)的轻量级和最小侵入性编程；
2. 通过依赖注入和面向接口实现松耦合；
3. 基于切面和惯例进行声明式编程；
4. 通过切面和模版减少样板式代码。

## (2)依赖注入
__DI(Dependency Injection)__
按照传统做法，每个对象负责管理与自己协作的对象(即它所依赖的对象)的引用，这样会导致__高度耦合__和__难以测试__的代码。

这时候，依赖注入的优势就体现出来了——__松耦合__，__依赖注入会将所依赖的关系自动交给目标对象，而不是让对象自己去获取依赖__。

创建应用程序组件之间协作的行为通常称为__装配(wiring)__。Spring有多种装配bean的方式，如xml描述配置、java描述配置。
##(3)应用切面
__AOP(Aspect-Oriented Programming)__
面向切面编程促使软软件系统能够实现关注点的分离。理想的系统应该由许多组件组成，每一个组件各负责一块特定内容，不应该承担额外的职责，例如日志、事务管理和安全这样的系统服务。事实上，这些系统服务应该成为横切关注点，因为它们会跨越系统的多个组件。

__如果将这些关注点分散到多个组件中，会有以下弊端：__
1. 实现系统关注点功能的代码会重复出现在多个组件中，如果你要修改关注点的逻辑，那么你就要修改各个模块中的相关实现。即时你把这些关注点抽象为一个独立的模块，其他模块只是调用它的方法，但方法的调用还是会重复出现在各个模块中。
2. 组件会因为那些与自身核心业务无关的代码而变得混乱。

AOP所带来的优势——__高内聚__,组件更加关注自身的业务，完全不需要了解涉及系统服务所带来的复杂性。

##(4)使用模版消除样板式代码
这没什么好讲的，显而易见。

# 二、容纳你的Bean
## (1)简述
__Spring容器__是Spring框架的核心，它使用DI(依赖注入)管理构成应用的组件，它会创建相互协作的组件之间的关联。
Spring自带多个容器实现，可以归结为两种类型：
1. __bean工厂__(由org.springframework.beans.factory.beanFactory接口定义)
2. __应用上下文__(由org.springframework.context.ApplicationContext接口定义)

由于bean工厂对于大多数应用来说往往太低级了，因此，上下文要比bean工厂更受欢迎。因此我们把注意力放在应用上下文上。

##(2)使用应用上下文
1. __AnnotationConfigApplicationContext__:从一个或多个基于Java的配置类中加载Spring应用上下文。
2. __AnnotationConfigWebApplicationContext__:从一个或多个基于Java的配置类中加载Spring Web应用上下文。
3. __ClassPathXmlApplicationContext__:从类路径下的一个或多个XM配置文件中加载上下文定义，把应用上下文作为类资源。
4. __FileSystemXmlApplicationContext__:从文件系统下的一个或多个XML配置文件中加载上下文定义。
5. __XmlWebApplicationContext__:从Web应用下的一个或多个XML配置文件中加载上下文定义。
应用上下文准备就绪之后，我们就可以调用上下文的getBean()方法从Spring容器中获取bean。
##(3)bean的生命周期
1. Spring对bean进行__实例化__。

2. Spring将__值__和bean的__引用__注入到bean对应的__属性__中。

3. 如果bean实现了__BeanNameAware接口__，Spring将bean的ID传给__setBeanName()__方法。

4. 如果bean实现了__BeanFactroryAware接口__，Spring将调用__setBeanFactory()__方法，将BeanFactory容器实例传入。

5. 如果bean实现了__ApplicationContextAware接口__，Spring将调用__setApplicationContext()__方法，将bean所在的应用上下文的引用传入。

6. 如果bean实现了__BeanPostProcessor接口__，Spring将调用它们的__postProcessBeforeInitialization()__方法。

7. 如果bean实现了__InitializingBean接口__，Spring将调用他们的__afterPropertiesSet()__方法。类似地，如果bean使用__init-method声明__了初始化方法，该方法也会被调用。

8. 如果bean实现了__BeanPostProcessor接口__，Spring将调用它们的__postProcessAfterInitialization()__方法。

9. 此时bean已经准备就绪，可以被应用程序使用了，它们将一直驻留在应用上下文中，直到该应用上下文被销毁。

10. 如果bena实现了__DisposableBean接口__，Spring将调用它的__destroy()__方法。同样，如果bean使用__destroy-method声明__了销毁方法，该方法也会被调用。

# 三、俯瞰Spring风景线
## (1)Spring模块
1. __Spring核心容器__：、
Bean、Core、Context、Expression、Context support。

__*所有的Spring模块都构建于核心容器之上。当你配置应用时，其实你隐式地使用了这些类。*__

2. __面向切面编程__：
AOP、Aspects。

3. __数据访问与集成__：
JDBC、Transaction、ORM、OXM、Message、JMS。

Springd ORM(Object-Relational Mapping)模块建立在对DAO的支持之上，并为多个ORM框架提供了一种构建DAO的简便方式。__Spring没有尝试去创建自己的ORM解决方案，而是对许多流行的ORM框架进行了集成__，包括Hibernate、Java Persisternce API、Java Data Object和iBATIS SQL Maps。__Spring的事务管理支持所有的ORM框架以及JDBC。__

4. __Web与远程调用__：
Web、Web servlet、Web portlet、WebSocket。

虽然Spring能够与多种流行的MVC框架进行集成，但它的Web和远程调用模块自带了与__一个强大的MVC框架——Sping MVC__，有助于在Web层提升应用的松耦合水平。
除了面向用户的Web应用，改模块还提供了多种构建与其他应用交互的远程调用方案。Spring远程调用功能集成了RMI(Remote Method Invocation)、Hessian、Burlap、JAX-WS， 同时Spring还自带了__一个远程调用框架——HTTPinvoker__。Spring还提供了暴露和使用__REST API__的良好支持
5. __Instrumentation__:
Instrument、Instrument Tomcat。

Spring的Instrumentation模块提供了__为JVM添加代理(agent)__的功能。具体来讲，它为Tomcat提供了一个织入代理，能够为Tomcat传递类文件，就像这些类文件是被类加载器加载的一样。

6. __测试__:
Test。

## (2)Spring Portfolio
__一些构建于核心Spring框架之上的框架和类库：__
* Spring Web Flow
基于流程的会话式Web应用
* Spring Web Service 
将Spring bean以声明的方式发布Web服务
* Spring Security 
安全机制
* Spring integration
与其他应用交互的通用集成模式的Spring声明式风格实现
* Spring Batch
对数据进行大量操作
* Spring Data
数据库
* Spring Social
社交网络扩展模块
* Spring Mobile
移动应用，移动Web应用开发
* Spring for Android
Android设备本地应用(原生应用)的简单支持
* Spring Boot
简化编程任务，致力于简化Spring本身