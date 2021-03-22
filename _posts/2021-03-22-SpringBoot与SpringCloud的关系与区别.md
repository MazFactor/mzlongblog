---
layout: post
title: SpringBoot与SpringCloud的关系与区别
categories: Java
description: SpringBoot与SpringCloud的关系与区别
keywords: SpringBoot SpringCloud
---
#### 一、SpringBoot和SpringCloud简介

1、SpringBoot：是一个快速开发框架，通过用MAVEN依赖的继承方式，帮助我们快速整合第三方常用框架，完全采用注解化（使用注解方式启动SpringMVC），简化XML配置，内置HTTP服务器（Tomcat，Jetty），最终以Java应用程序进行执行。

2、SpringCloud： 是一套目前完整的微服务框架，它是是一系列框架的有序集合。它只是将目前各家公司开发的比较成熟、经得起实际考验的服务框架组合起来，通过SpringBoot风格进行再封装屏蔽掉了复杂的配置和实现原理，最终给开发者留出了一套简单易懂、易部署和易维护的分布式系统开发工具包。它利用Spring Boot的开发便利性巧妙地简化了分布式系统基础设施的开发，如服务发现注册、配置中心、消息总线、负载均衡、断路器、数据监控等，都可以用SpringBoot的开发风格做到一键启动和部署。

#### 二、SpringBoot和SpringCloud的关系与区别

1、SpringBoot只是一个快速开发框架，使用注解简化了xml配置，内置了Servlet容器，以Java应用程序进行执行。

2、SpringCloud是一系列框架的集合，可以包含SpringBoot。

#### 三、SpringBoot是微服务框架吗？

1、SpringBoot只是一个快速开发框架，算不上微服务框架。

2、SpringCloud+SpringBoot 实现微服务开发。具体的来说是，SpringCloud具备微服务开发的核心技术：RPC远程调用技术；SpringBoot的web组件默认集成了SpringMVC，可以实现HTTP+JSON的轻量级传输，编写微服务接口，所以SpringCloud依赖SpringBoot框架实现微服务开发。

四、SpringMVC在3.0开始支持采用注解方式启动，所以可以不再配置传统的XML配置文件。

<br>
原文链接：[https://blog.csdn.net/weixin_42315600/article/details/84134094](https://blog.csdn.net/weixin_42315600/article/details/84134094)

