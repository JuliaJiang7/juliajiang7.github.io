---
title: SpringBoot-AOP实现数据权限管理
date: 2021-03-07 14:58:01
tags:
  - Java
  - SpringBoot
  - AOP
categories: SpringBoot

typora-copy-images-to: ..\pictures
---

在系统开发中，总有这样的需求：控制登陆用户所能看到的数据，即进行数据权限管理。比如，用户查询所有员工数据时，只能展示该用户有权限查看的部门和业务线数据。当然，这可以在 sql 中进行限制，但是这样的数据权限处理就不存在通用性。

实现数据权限管理，最核心的问题在于通用且不影响原有的业务代码，即 **高内聚低耦合**。这种场景可以使用AOP，将数据权限代码抽离出来，在需要查询数据时限制查询结果，从而在不修改业务代码的情况下实现方法增强。

本文将以一个小例子介绍如何使用AOP实现数据权限管理，源码：[data-permission-aop-demo](https://github.com/JuliaJiang7/data-permission-aop-demo)

<!--more-->

## 1. 功能需求

用户登录后，查看所有员工信息时，只展示有权限查看的数据。权限的限制维度是部门id和业务线id，即登录用户展示的数据是同时满足部门id和业务线id的员工信息。

## 2. 流程图

<img src="/pictures/Spring-AOP流程.jpg"/>

## 3. 类图

<img src="/pictures/Spring-AOP类图.jpg"/>

## 4. 数据库

<img src="/pictures/AOP-数据库UML.png"/>

## 5. 源码

[data-permission-aop-demo](https://github.com/JuliaJiang7/data-permission-aop-demo)



## 6. 参考引用

1. [SpringBoot-AOP处理数据过滤](http://wjwcloud.com/springboot/2019/03/01/SpringBoot_AOP_dataAuthority/)