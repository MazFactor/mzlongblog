---
layout: post
title: Idea 解决 Could not autowire. No beans of 'xxxx' type found 的错误提示
categories: Java
description: Idea 解决 Could not autowire. No beans of 'xxxx' type found 的错误提示
keywords: Java Idea
---
IntelliJ Idea 解决 Could not autowire. No beans of ‘xxxx' type found 的错误提示

哈，在使用 @Autowired 时，今天又遇一坑，这俩波浪线是干啥子嘛：

![讨厌的波浪线]({{assets_base_url}}/images/blog/其它/讨厌的波浪线.jpg)
<center>
<div style="color:orange; border-bottom: 1px solid #d9d9d9;display: inline-block;color: #999;padding: 2px;">讨厌的波浪线</div>
</center>

然鹅，试了一下，控制台也不报错，可以正常运行：

![控制台输出正常]({{assets_base_url}}/images/blog/其它/控制台输出正常.jpg)
<center>
<div style="color:orange; border-bottom: 1px solid #d9d9d9;display: inline-block;color: #999;padding: 2px;">控制台输出正常</div>
</center>

数据也有：

![数据正常]({{assets_base_url}}/images/blog/其它/数据正常.jpg)
<center>
<div style="color:orange; border-bottom: 1px solid #d9d9d9;display: inline-block;color: #999;padding: 2px;">数据正常</div>
</center>

于是，又再百度上找答案。。

### 问题分析
在 Idea 的 spring 工程里，经常会遇到 Could not autowire. No beans of ‘xxxx' type found 的错误提示。但程序的编译和运行都是没有问题的，这个错误提示并不会产生影响。但红色的错误提示在有些有强迫症的程序员眼里，多多少少有些不太舒服。

#### 问题原因其一
第一个是 Intellij IDEA 本身工具的问题。

解决办法：

（1）不理它。

（2）在注解上加上：

```java	
@Autowired(required = false)
```
（3）降低 Autowired 检测的级别，将 Severity 的级别由之前的 error 改成 warning 或其它可以忽略的级别。

#### 还有一个原因

这个博主没有遇到，友情粘贴！

第二个原因便是我们导入 @Service 包的时候导入包错误造成的。

spring auto scan 配置，在编辑情况下，无法找不到对应的bean，于是提示找不到对应 bean 的错误。常见于 mybatis 的 mapper，如下：

```xml
<!-- mapper scanner configurer -->
<bean id="mapperScannerConfig" class="org.mybatis.spring.mapper.MapperScannerConfigurer">
  <property name="basePackage" value="com.adu.spring_test.mybatis.dao" />
  <property name="sqlSessionFactoryBeanName" value="sqlSessionFactory" />
</bean>
```

解决办法：
	
> 错误导包 import com.alibaba.dubbo.config.annotation.Service;

正确的包应该是下面这个:

```java	
import org.springframework.stereotype.Service;
```

转载于：[https://www.jb51.net/article/154488.htm](https://www.jb51.net/article/154488.htm)