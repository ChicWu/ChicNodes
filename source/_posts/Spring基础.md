---
title: Spring基础
date: 2018-09-06 16:48:45
top: 9
tags:
 spring
---

　　Spring框架是一个 轻量级的企业级开发的站式解决方案。 所谓解决方案就是可以基于Spring解决Java EE开发的所有问题。Spring框架主要提供了IoC 容器、AOP、数据访问、Web开发、消息、测试等相关技术的支持。
　　Spring使用简单的POJO (Plain Old Java Object,即无任何限制的普通Java对象)来进行企业级开发。每个被Spring管理的Java对象都称之为Bean;而Spring提供了一个IoC容器用来初始化对象，解决对象间的依赖管理和对象的使用。
<!-- more -->
# 概述
　　Spring框架是一个 轻量级的企业级开发的站式解决方案。 所谓解决方案就是可以基于Spring解决Java EE开发的所有问题。Spring框架主要提供了IoC 容器、AOP、数据访问、Web开发、消息、测试等相关技术的支持。
　　Spring使用简单的POJO (Plain Old Java Object,即无任何限制的普通Java对象)来进行企业级开发。每个被Spring管理的Java对象都称之为Bean;而Spring提供了一个IoC容器用来初始化对象，解决对象间的依赖管理和对象的使用。
　　Spring框架本身有四大基本原则：
* `使用POJO进行轻量级和最小侵入式开发`
* `通过依赖注入和基于接口编程实现松耦合`
* `通过AOP和默认习惯进行声明式编程`
* `使用AOP和模版减小模版式代码`

## Spring的模块
　　spring是模块化的，这就意味着你可以只使用你需要的spring模块。spring的模块如下所示：
![Spring模块](/img/Spring模块.png)
### 核心容器
* Sning-Core：核心工具类，Sping其他模块大最使用SingCcre;
* Spring-Beans：Spring 定义Bean的支持;
* Spring-Context：运行时Spring容器;
* Spring-Context-Support：Spring容器对第三方包的集成支持;
* Spring Expression：使用表达式语言在运行时查询和操作对象;

### AOP
* Spring-AOP：基于代理的AOP支持;
* Spring-Aspects：基于AspeetU的AOP支持。

### 消息
* Spring Messaging：对消息架构和协议的支持。

### 数据访问/集成
* Spring-Web：提供基础的Web集成的功能，在Web项目中提供Spring的容器;
* Spring Webmve：提供基于Servlet的Spring MVC;
* Spring WebSocket：提供WebSocket功能;
* Spring Webmve Porlet：提供Portlet环境支持。

### Web
* SpingJDBC：提供以IDBC访问数据库的支持;
* Sping-TX：提供编程式和声明式的事务支持;
* Spring-ORM:提供对对象关系映射技术的支持;
* SpingOXM:提供对对象人xml映射技术的支持;Snig-M提供对IMS的支持。


# 基础配置
　　spring进行配置的方式本有三种：配置文件方式，注解方式和java类方式，但是由于现在较为常用的是使用springBoot进行开发提倡零配置约定大于配置默认不在有web.xml等配置文件，所以不再考虑配置文件方式。
## 注解方式
### 配置包扫描
### Bean的实例化方法
#### 构造方法
#### 作用域
#### 声明周期
### Bean的属性注入
#### 构造器注入
#### Setter方法注入
#### 集合属性注入
## java类方式

# IOC
　　我们经常说的控制翻转(Inversion of Control-I0C)和依赖注入(dependency injecton-D)在Spring环境下是等同的概念，控制翻转是通过依赖注入实现的。所谓依赖注入指的是容基负责创建对象和维护对象的依赖关系，而不是通过对象本身负责自己的创建和解决自己的依赖。
　　依赖注入的主要目的是为了解耦，体现了种“组合”的理念。如果你希望你的类具各某项功能的时候，是继承自一个具有此功能的父类好呢?还是组合另外一个具有这个功能的类好呢?答案是不言而喻的，继承一个父类，子类将与父类耦合，组合另外一个类则使耦合度大大降低。
　　Spring IoC容器(ApplicationContext)负责创建Bcan，并通过容器将功能类Bean注入到你需要的Bean中。Spring 提供使用xml、注解、Java 配置、groovy 配置实现Bean的创建和注入。
　　`声明Bean的注解:`
* @Component组件，没有明确的角色,
* @Service在业务逻辑层( service层)使用,
* @Repository在数据访问层(dao层)使用,
* @Conroller在展现层(MVC- Spring MVC)使用,

　　`注入Bean的注解,一般情况下通用`
* @Autowired: Spring 提供的注解,
* @Inject: JSR-330提供的注解,
* @Resource: JSR 250提供的注解,

　　@Autowired. @Inject. @Resurce可注解在se方法上或者属性上，建议注解在属性上，优点是代码更少、层次更清晰。


# AOP
　　AOP为Aspect Oriented Programming的缩写，意为：面向切面编程，通过预编译方式和运行期动态代理实现程序功能的统一维护的一种技术。AOP是OOP的延续，是软件开发中的一个热点，也是Spring框架中的一个重要内容，是函数式编程的一种衍生范型。利用AOP可以对业务逻辑的各个部分进行隔离，从而使得业务逻辑各部分之间的耦合度降低，提高程序的可重用性，同时提高了开发的效率。
　　AOP是一个概念，并没有设定具体语言的实现，它能克服那些只有单继承特性语言的缺点，spring2.0之后整合AspectJ第三方AOP技术。AspectJ是一个面向切面的框架，它扩展了Java语言。AspectJ定义了AOP语法所以它有一个专门的编译器用来生成遵守Java字节编码规范的Class文件。
## 主要功能
　　日志记录，性能统计，安全控制，事务处理，异常处理等等
## 主要意图
　　将日志记录，性能统计，安全控制，事务处理，异常处理等代码从业务逻辑代码中划分出来，通过对这些行为的分离，我们希望可以将它们独立到非指导业务逻辑的方法中，进而改变这些行为的时候不影响业务逻辑的代码。
## AOP底层实现
　　AOP分为静态AOP和动态AOP。静态AOP是指AspectJ实现的AOP，他是将切面代码直接编译到Java类文件中。动态AOP是指将切面代码进行动态织入实现的AOP。Spring的AOP为动态AOP，实现的技术为： JDK提供的动态代理技术 和 CGLIB(动态字节码增强技术)。
### JDK动态代理和CGLIB动态代理
* jdk的动态代理只可以为接口去完成操作，而cglib它可以为没有实现接口的类去做代理，也可以为实现接口的类去做代理。
* 如果目标对象，有接口，优先使用jdk动态代理；如果目标对象，无接口，使用cglib动态代理。

## AOP开发

# FAQ
### IOC和DI区别?
　　IOC 控制反转，是指对象实例化权利由spring容器来管理
　　DI 依赖注入 在spring创建对象的过程中，对象所依赖的属性通过配置注入对象中，在Spring环境下是等同的概念。
### ApplicationContext与BeanFactory关系
![](/img/ApplicationContext与BeanFactory关系.png)
　　ApplicationContext它是扩展BeanFactory接口。BeanFactory它采取延迟加载的方案，只有真正在getBean时才会实例化Bean,在开发中我们一般使用的是ApplicationContext,真正使用的是其实现类AppliCationContext它会在配置文件加载时，就会初始化Bean,并且ApplicationContext它提供不同的应用层的Context实现。例如在web开发中可以使用WebApplicationContext
　　`FileSystemXmlAppliCationContext 根据文件路径获取`
　　`ClassPathXmlApplicationContext  根据类路径获取`



