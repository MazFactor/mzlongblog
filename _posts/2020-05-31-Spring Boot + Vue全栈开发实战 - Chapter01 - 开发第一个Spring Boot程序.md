---
layout: post
title: Spring Boot + Vue全栈开发实战 - Chapter01 - 开发第一个Spring Boot程序
categories: Spring Boot
description: Spring Boot + Vue全栈开发实践
keywords: Springboot 实战
---
Spring 作为一个轻量级的容器，在Java EE 开发中得到了广泛的应用，但是Spring 的配置烦琐雕肿，在和各种第三方框架进行整合时代码量都非常大，并且整合的代码大多是重复的，为了便开发者能够快速上手Spring，利用Spring 框架快速搭建Java EE 项目， Spring Boot 应运而生。

Spring Boot 带来了全新的自动化配置解决方案，使用Spring Boot 可以快速创建基于Spring 生产级的独立应用程序。Spring Boot 中对一些常用的第三方库提供了默认的自动化配置方案，使得开发者只需要很少的Spring 配置就能运行一个完整的Java EE 应用。Spring Boot 项目可以采用传统的方案打成war 包，然后部署到Tomcat 中运行。也可以直接打成可执行jar 包，这样通过java -jar命令就可以启动一个Spring Boot 项目。总体来说， Spring Boot 主要有如下优势：

* 提供一个快速的Spring项目搭建渠道。
* 开箱即用，很少的Spring配置就能运行一个Java EE项目。
* 提供了生产级的服务监控方案。
* 内嵌服务器，可以快速部署。
* 提供了一系列非功能性的通用配置。
* 纯Java配置，没有代码生成，也不需要XML配置。

Spring Boot 是一个“年轻”的项目，发展非常迅速，特别是在Spring Boot 2.0之后，许多API都有较大的变化，本书的写作基于目前最新的稳定版2.0.4（本书写作时的最新版），因此需要Java 8 或9 以及Spring Framework 5.0.8.RELEASE或更高版本，同时，构建工具的版本要求为Maven 3.2 + 或Gradle 4。

### 创建Maven工程
Spring Boot 工程可以通过很多方式创建，最通用的方式莫过于使用Maven，因为大多数的IDE都支持Maven。在Eclipse、Intellij Idea中创建Maven工程很常见也很容易掌握，通过spring.io同Idea，这里都不再赘述。以下是使用命令创建Maven工程。

首先定位到目标目录，在cmd窗口中执行如下命令：

```dos
mvn archetype:generate -DgroupId=org.sang -DartifactId=chapter01 -DarchetypeArtifactId=maven-archetype-quickstart -DinteractiveMode=false
```

上述命令解释：

* -DgroupId 组织Id（项目包名）
* -DartifactId ArtifactId（项目名称或才模块名称）
* -DarchetypeArtifactId 项目骨架
* -DinteractiveMode 是否使用交互模式

使用命令将项目创建好之后，直接用Eclipse或者Intellij IDEA打开即可。注意，用IDE打开新创建好的项目之后，仍需要配置一下Maven的路径，否则会产生jar包依赖问题。

### 项目构建
#### 添加依赖
首先添加spring-boot-starter-parent作为parent，代码如下：

```java
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>srping-boot-starter-parent</artifactId>
    <version>2.0.4.RELEASE</version>
</parent>
```

spring-boot-starter-parent 是一个特殊的Starter， 提供了一些Maven 的默认配置，同时还提供了dependency-management，可以便开发者在引入其他依赖时不必输入版本号，方便依赖管理。SpringBoot 中提供的Starter非常多，这些Starter 主要为第三方库提供自动配置，例如要开发一个Web项目，就可以先引入一个Web 的Starter ，代码如下：

```java
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
</dependencies>
```

#### 编写启动类
接下来创建项目的入口类，在Maven工程的java目录下创建项目的包，包里创建一个App类，代码如下：

```java
@EnableAutoConfiguration
public class App {
    public static void main(String[]) args {
        SpringApplication.run(App.class, args);
    }
 }

代码解释：

* @EnableAutoConfiguration 注解表示开户自动化配置。由于项目中添加了 spring-boot-starter-web 依赖，因此在开户了自动化配置之后 会自动进行 Spring 和 SpringMVC 的配置。

*在 Java 项目的 main 方法中，通过 SpringApplication 中的 run 方法启动项目。第一个参数传入App.class，告诉Spring哪个是主要组件。第二个参数是运行时输入的其他参数。

接下来创建一个SpringMVCk中的控制器 —— HelloController，代码如下：

```java
@RestController
public class HelloController {
    @GetMapping("/hello")
    public String hello() {
        return "hello spring boot";
    }
}

在控制器中提供了一个“/hello”接口，此时需要配置包扫描才能将HelloController注册到SpringMVC容器中，因此在App类上面再添加一个注解@ComponentScan进行包扫描，代码如下：

```java
@EnableAutoConfiguration
@ComponentScan
public class App {
    public static void main(String[] args) {
        SpringApplication.run(App.class, args);
    }
}    
```

也可以直接使用组合注解@Spring BootApplication来代替@EnableAutoConfiguration 和 @ComponentScan，这里为了方便理解与学习，仍使用@EnableAutoConfiguration 和@ComponentScan。

### 项目启动与部署
同样为了本着一致的原则，我们继续使用命令的方式来启动和打包部署项目。

#### 启动
定位到项目根目录，使用Maven的如下命令即可启动项目：

```dos
mvn spring-boot:run
```

启动成功后，可以在浏览器中直接访问/hello接口。如下图所示：

![访问/hello]({{assets_base_url}}/images/blog/SpringBoot+Vue全栈开发实战/chapter01/访问接口hello.jpg)
<center>
<div style="color:orange; border-bottom: 1px solid #d9d9d9;display: inline-block;color: #999;padding: 2px;">访问/hello</div>
</center>

#### 打包
Spring Boot 应用也可以直接打成jar包运行。在生产环境中，也可以通过这样的方式来运行一个Spring Boot应用。要将Spring Boot 打成jar包运行，首先需要添加一个plugin到pom.xml文件中，代码如下：

```java
<build>
    <plugins>
        <plugin>
            <groupId>org.springframework.boot<groupId>
            <artifactId>spring-boot-maven-plugin</artifactId>
        </plugin>
    </plugin>
</buld>
```

然后运行mvn命令进行打包，代码如下：

```dos
mvn package
```

打包成功后在项目的target目录下会生成一个jar文件，通过java -jar命令直接启动这个jar文件即可。最后附上项目最终结构如下：

![项目结构]({{assets_base_url}}/images/blog/SpringBoot+Vue全栈开发实战/chapter01/项目结构.jpg)
<center>
<div style="color:orange; border-bottom: 1px solid #d9d9d9;display: inline-block;color: #999;padding: 2px;">项目结构</div>
</center>