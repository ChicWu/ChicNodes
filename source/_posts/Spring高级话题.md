---
title: Spring高级话题(二);
date: 2018-09-06 16:49:21
tags:
	spring
---
　　这家伙很懒竟然没写摘要
<!-- more -->
# spring事务管理
　　spring事务有四个优点：
* 提供一致的对于不同的事务管理的API
* 支持声明式事务管理(重点)
* 编程事务管理(在开发中应用比较少)
* 优秀的整合与Spring的数据访问
## 事务管理的API
### 事务的传播机制
## 声明式事物
### 开启声明式事务
spring开启声明式事务：
在application中
springBoot开启声明式事务：
，首先使用注解 `@EnableTransactionManagement`开启事务支持后，然后在访问数据库的Service方法上添加注解`@Transactional`便可。
# SpringAOP编程
# Spring jdbc Template