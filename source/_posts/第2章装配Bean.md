---
title: 第2章装配Bean
date: 2016/11/21 20:46:25
updated: 2016/11/21 20:46:25
categories:
- Spring实战第4版读书笔记
tags:
- Spring
---
# 一、简述
Spring提供了三种主要的装配机制：
1. 在XML中进行显示配置。
2. 在Java中进行显示配置。
3. 隐式的bean发现机制和自动装配。
推荐：自动配置机制>JavaConfig>XML

# 二、装配方式

## (1)自动化装配bean
Spring从两个角度来实现自动化装配:
* 组件扫描(component scanning)：Spring会自动发现应用上下文中所创建的bean。
* 自动装配(autowiring)：Spring会自动满足bean中间的依赖。

### 组件扫描
__@Component注解__表明该类会作为组件类；
__@ComponentScan注解__能够在Spring中启用组件扫描，默认扫描该类所在包以及这个包的所有子包。（也可以用XML来启用扫描）

### 自动装配
__@Autowired注解__表明当Spring创建这个bean的时候，会通过构造器来进行实例化并且会传入一个可设置给所需的对应类型的bean。
这个注解不仅可以用在构造器上，也可以用在属性的Setter方法上和其他的方法上。
__@Inject注解__来源于Java依赖注入规范，在大多数情景下和@AutoWired可以相互替换。

__注意：__
* 如果没有匹配的bean，那么在应用的上下文创建的时候，Spring会抛出一个异常。为了避免抛出异常，你可以将required的属性设置为false，但是要注意进行null检查；
* 如果有多个bean都能满足依赖关系的画，Spring将会抛出一个异常，表明没有明确指出要选择哪个bean进行自动装配。在之后的文章中我们再来讨论自动装配中的歧义性。

## (2)通过Java代码装配bean

### 创建配置类
__@Configuration注解__表明该类是一个配置类，该类应该包含在Spring应用上下文中如何创建bean的细节。

### 声明简单的bean
__@Bean注解__会高速Spring这个方法将会返回一个对象，该对象要注册为Spring应用上下文中的bean。默认情况下，beand的ID和带有@Bean注解的方法名一样。

### 借助JavaConfig实现注入
最简单的方式就是引用创建bean的方法。
同样使用@Bean注解，这表明这个方法会创建一个bean实例并将其注册到Spring应用上下文中
构造器和Setter方法只是@Bean的两个简单样例，带有@Bean注解的方法可以采用任何必要的Java功能来产生bean实例。

## (3)通过XML代码装配bean
1. <beans>对应注解@Configuration
![最为简单的Spring XML配置](http://upload-images.jianshu.io/upload_images/3828003-61b64dd310e24713.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
2. <bean> 对应注解@Bean
<bean id="id名" class="包名.类名">
3. 构造器注入初始化bean有两种基本的配置方案：
* <constructor-arg>元素

* 使用Spring3.0所引入的c-命名空间

（1）注入bean引用：
![<constructor-arg>元素构造器注入.png](http://upload-images.jianshu.io/upload_images/3828003-ad184ad1fc6ec671.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![引入c-命名空间](http://upload-images.jianshu.io/upload_images/3828003-e6dd3b7cad7f4dac.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


![c-命名空间构造器注入](http://upload-images.jianshu.io/upload_images/3828003-648082d8a5d3bfa1.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


![c-命名空间构造器注入解释](http://upload-images.jianshu.io/upload_images/3828003-193c16d25809d8e3.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![数字索引参数](http://upload-images.jianshu.io/upload_images/3828003-80c75d85d9dcb9f8.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

（2）字面量注入到构造器


![<constructor-arg>元素注入](http://upload-images.jianshu.io/upload_images/3828003-5d884dc4bcd00b41.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


![c-命名空间注入](http://upload-images.jianshu.io/upload_images/3828003-c5209294768b810c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

（3）装配集合
这种情况是<constructor-arg>能够实现，c-命名控件却无法做到的。 

![<list>是<constructor-arg>的子元素](http://upload-images.jianshu.io/upload_images/3828003-816a0ebb89d77e80.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

<list>也可以用<set>，区别是<set>所有重复的值会被忽略，存放顺序也不会得以保证。
<value>也可以用<ref>来表示存放的是引用列表。
4. 属性注入（和构造器注入类似）
* <property>元素注入
* p-命名空间注入

（1）设置属性
![<property>元素注入](http://upload-images.jianshu.io/upload_images/3828003-068e8797150fea8e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


![p-命名空间引入](http://upload-images.jianshu.io/upload_images/3828003-7c1aa1eae52091ee.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


![p-命名空间注入](http://upload-images.jianshu.io/upload_images/3828003-d838064f8f452802.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


![p-命名空间注入解释](http://upload-images.jianshu.io/upload_images/3828003-3120b45a9fae5d7d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

（2）字面量注入属性

![字面量注入属性](http://upload-images.jianshu.io/upload_images/3828003-eaa3569a54d182dd.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## (4)导入和混合配置
1. 两个JavaConfig组合
@Import注解：
@Import({AConfig.class,BConfig.class})
2. 在JavaConfig中引用XML配置
@Import注解+@ImportResource注解：
@Import(AConfig.class)
@ImportResource("classpath:b-config.xml")
3. 在XML配置中引用JavaConfig
<import>元素只能导入其他的XML配置；<bean>元素可以将JavaConfig类导入到XML配置中：
<import resource="a-config.xml">
<bean class="包名.配置类名">