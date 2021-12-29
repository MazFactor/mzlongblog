---
layout: post
title: Spring Boot + Vue全栈开发实战 - Chapter03 - Spring Boot整合Web开发
categories: SpringBoot
description: Spring Boot + Vue全栈开发实践
keywords: Springboot 实战
---
Spring Boot整合 Web 开发这一章节中主要涉及依赖项 spring-boot-starter-web，并且对其进行定制化，去除默认的json处理器 jackson-databind，最终形成一套基于 Spring Boot 的可返回 json 格式的项目框架。

首先，需要同[chapter02 - SpringBoot基础配置](https://www.baidu.com)一样创建一个多模块项目，父项目为 SpringBootLearn，但子项目改为 chapter03。项目最终目录结构如下：

![项目结构]({{assets_base_url}}/images/blog/SpringBoot+Vue全栈开发实战/chapter03/项目结构.jpg)

### Jackson
JSON 是目前主流的前后端数据传输方式， Spring MVC 中使用消息转换器 HttpMessageConverter 对 JSON 的转换提供了很好的支持，在 Spring Boot 中更进一步，对相关配置做了更进一步的简化。默认情况下，当开发者新创建一个 Spring Boot 项目后，添加 Web 依赖，代码如下：

```java
<dependency>
<groupid>org.springframework.boot</groupid>
<artifactid>spring-boot-starter-web</artifactid>
</dependency>
```

spring-boot-starter-web 默认依赖于 jackson-databind，将 jackson-databind 作为JSON处理器，此时不需要添加额外的JSON转换器就能返回一段JSON格式数据。如创建一个Book实体类：

```java
public class Book {
    private String name;
    private String author;
    @JsonIgnore
    private Float price;
    @JsonFormat(pattern = "yyyy-MM-dd")
    private Date publicationDate;
    // 省略Getter/Setter
}
```

然后创建BookController返回Book对象：

```java
@RestController
public class BookController {
    @GetMapping("/book")
    public Book book() {
        Book book = new Book();
        book.setAuthor("罗贯中");
        book.setName("三国演义");
        book.setPrice(30.0f);
        book.setPublicationDate(new Date());
        return book;
    }
}
```

由于我们的application.yml中的配置如下：

```xml
server:
  port: 8081
  servlet:
    context-path: /chapter03
  tomcat:
    uri-encoding: utf-8
spring:
  http:
    encoding:
      force-response: true
```

所以在浏览器中输入“http://localhost:8081/chapter03/book”即可看到返回的JSON数据，如下图所示：

![默认集成jackson返回结果]({{assets_base_url}}/images/blog/SpringBoot+Vue全栈开发实战/chapter03/默认集成jackson返回结果.jpg)

这是Spring Boot 自带的处理方式，如果采用这种方式，对于字段忽略、日期格式化等常见需求都可以通过注解来解决。通过Spring中默认提供的MappingJackson2HttpMessageConverter来实现，当然这里可以根据实际需要自定义JSON转换器。

### 返回JSON数据
#### Gson
常见的JSON处理器除了jackson-databind之外还有Google的开源框架Gson和阿里的Fastjson。在使用Gson或Fastjson之前需要除去默认的jackson-databind，然后引入Gson或者Fastjosn依赖。

```xml
<dependency>
    <groupId>org.springfrarnework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
    <exclusions>
        <exclusion>
            <groupId>com.fasterxml.jackson.core</groupId>
            <artifactId>jackson-databind</artifactId>
        </exclusion>
    </exclusions>
</dependency>
<dependency>
    <groupId>com.google.code.gson</ groupId>
    <artifactId>gson</artifactId>
</dependency>
```

由于Spring Boot 中默认提供了Gson 的自动转换类GsonHttpMessageConvertersConfiguration,因此Gson 的依赖添加成功后， 可以像使用jackson-databind 那样直接使用Gson 。但是在Gson 进行转换时， 如果想对日期数据进行格式化， 那么还需要开发者自定义HttpMessageConverter 。自定义HttMessageConverter 可以通过如下方式。首先看GsonHttpMessageConvertersC onfiguration 中的一段源码：

```java
@Bean
@ConditionalOnMissingBean
public GsonHttpMessageConverter gsonHttpMessageConverter(Gson gson) {
    GsonHttpMessageConverter converter = new GsonHttpMessageConverter();
    converter.setGson(gson);
    return converter;
}
```

@ConditionalOnMissingBean 注解表示当项目中没有提供GsonHttpMessageConve阳时才会使用默认的GsonHttpMessageConverter ， 所以开发者只需要提供一个GsonHttpMessageConverter 即可，代码如下：

```java
@Configuration
public class GsonConfig {
    @Bean
    GsonHttpMessageConverter gsonHttpMessageConverter() {
        GsonHttpMessageConverter converter = new GsonHttpMessageConverter();
        GsonBuilder builder = new GsonBuilder();
        builder.setDateFormat("yyyy-MM-dd");
        bulder.excludeFieldWithModifiers(Modifier.PROTECTED);
        Gson gson = builder.create();
        converter.setGson(gson);
        return converter;
    }
}
```

代码解释：

* 开发者自己提供一个GsonH即MessageConverter 的实例。
* 设直Gson 解析时日期的格式。
* 设直Gson 解析时修饰符为protected 的字段被过滤掉。
* 创建Gson 对象放入GsonHttpMessageConverter 的实例中并返回converter。

此时，将Book 类中的price 字段的修饰符改为protected。最后在浏览器中输入“http://localhost:8081/chapter03/book”即可看到返回的JSON数据，如下图所示：

![集成Gson返回结果]({{assets_base_url}}/images/blog/SpringBoot+Vue全栈开发实战/chapter03/集成Gson返回结果.jpg)

#### Fastjson
Fastjson 是阿里巴巴的一个开源JSON 解析框架，是目前JSON 解析速度最快的开源框架，该框架也可以集成到Spring Boot 中。不同于Gson, Fastjson 继承完成之后并不能立马使用，需要开发者提供相应的HttpMessageConverter 后才能使用，集成fastjson 的步骤如下。

首先除去jackson-databind依赖，引入fastjson依赖。但由于spring-boot-starter-web 依赖于spring-boot-starter-json，面后者又依赖于jackson-databind，此外还有jackson-datatype-jsr310、jackson-module-parameter-names、jackson-datatype-jdk8、jackson-annotations、jackson-core，既然不打算用jackson，干脆把这整个spring-boot-starter-json都去除掉，再引入fastjson，最终整个pom文件中配置如下：

```xml
<?xml version="1.0"?>
<project xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd" xmlns="http://maven.apache.org/POM/4.0.0"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance">
  <modelVersion>4.0.0</modelVersion>
  <parent>
    <groupId>SpringBootLearn</groupId>
    <artifactId>SpringBootLearn</artifactId>
    <version>1.0-SNAPSHOT</version>
  </parent>
  <groupId>chapter03</groupId>
  <artifactId>chapter03</artifactId>
  <version>1.0-SNAPSHOT</version>
  <name>chapter03</name>
  <url>http://maven.apache.org</url>
  <dependencies>
    <dependency>
      <groupId>junit</groupId>
      <artifactId>junit</artifactId>
      <version>3.8.1</version>
      <scope>test</scope>
    </dependency>
    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-web</artifactId>
      <exclusions>
        <exclusion>
          <groupId>org.springframework.boot</groupId>
          <artifactId>spring-boot-starter-json</artifactId>
        </exclusion>
      </exclusions>
    </dependency>
    <dependency>
      <groupId>com.alibaba</groupId>
      <artifactId>fastjson</artifactId>
      <version>1.2.47</version>
    </dependency>
  </dependencies>
</project>
```

然后配置fastjson 的HttpMessageConverter：

```java
@Configuration
public class MyFastJsonConfig {
    @Bean
    FastJsonHttpMessageConverter fastJsonHttpMessageConverter() {
        FastJsonHttpMessageConverter converter = new FastJsonHttpMessageConverter();
        FastJsonConfig config = new FastJsonConfig();
        config.setDateFormat("yyyy-MM-dd");
        config.setCharset(Charset.forName("UTF-8"));
        config.setSerializerFeatures(
            SerializerFeature.WriteClassName,
            SerializerFeature.WriteMapNullValue,
            SerializerFeature.PrettyFormat,
            SerializerFeature.WriteNullListAsEmpty,
            SerializerFeature.WriteNullStringAsEmpty
        );
        converter.setFastJsonConfig(config);
        return converter;
    }
}
```

代码解释：

* 自定义MyFastJsonConfig ，完成对FastJsoHttpMessageConverter Bean 的提供。
* 第7～ 15 行分别配置了JSON 解析过程的一些细节，例如日期格式、数据编码、是否在生成的JSON 中输出类名、是否输出value 为null 的数据、生成的JSON 格式化、空集合输出口而非null 、空字符串输出""而非null 等基本配置。

MyFastJsonConfig 配置完成后，还需要配置一下响应编码，否则返回的JSON 中文会乱码，在application. properties 或application.yml 中添加配置，这里以application.yml为例：

```xml
spring:
  http:
    encoding:
      force-response: true
```
在浏览器中输入“http://localhost:8081/chapter03/book”即可看到返回的JSON数据，如下图所示：

![集成Fastjson返回结果]({{assets_base_url}}/images/blog/SpringBoot+Vue全栈开发实战/chapter03/集成Fastjson返回结果.jpg)

对于FastJsonHttpMessageConverter 的配置，除了上面这种方式之外，还有另一种方式。在Spring Boot 项目中，当开发者引入spring-boot-starter-web 依赖之后，该依赖又依赖了spring-boot-autoconfigure ， 在这个自动化配置中，有一个WebMvcAutoConfiguration 类提供了对Spring MVC 最基本的配置， 如果某一项自动化配置不满足开发需求，开发者可以针对该项自定义配置，只需要实现WebMvcConfigurer 接口即可（在Spring 5.0 之前是通过继承WebMvcConfigurerAdapter 类来实现的），代码如下：

```java
@Configuration
public class MyWebMvcConfig Implements WebMvcConfigurer {
    @Override
    public void configureMessageConverters(List<HttpMessageConverter <?>> converters) {
        FastJsonHttpMessageConverter converter= new FastJsonHttpMessageConverter();
        FastJsonConfig config = new FastJsonConfig();
        config.setDateFormat("yyyy-MM-dd");
        config.setCharset (Charset.forName("UTF-8")) ;
        config.setSerializerFeatures(
        Seriali zerFeature.WriteClassName,
        SerializerFeature.WriteMapNullValue,
        SerializerFeature.PrettyFormat,
        SerializerFeature.Wri teNullListAsEmpty,
        SerializerFeature.WriteNullStringAsEmpty
        converter.setFastJsonConfig(config);
        converters.add(converter) ;
    }
}
```

代码解释：

* 自定义MyWebMvcConfig 类并实现WebMvcConfigurer 接口中的configureMessageConverters方法。
* 将自定义的FastJsonHttpMessageConverter 加入converters 中。

> 如果使用了Gson，也可以采用这种方式配置，但是不推荐。因为当项目中没有GsonHttpMessageConverter 时， Spring Boot 自己会提供一个GsonHttpMessageConverter，此时重写configureMessageConverters 方法， 参数converters 中已经有GsonHttpMessageConverter的实例了，需要替换已有的GsonHttpMessageConverter 实例，操作比较麻烦，所以对于Gson，推荐直接提供GsonHttpMessageConverter。

最后附上一张项目类图：
![项目涉及类图]({{assets_base_url}}/images/blog/SpringBoot+Vue全栈开发实战/chapter03/项目涉及类图.jpg)


### 静态资源访问
在SpringMVC中，对于静态资源都需要开发人员手动配置静态资源过滤。Spring Boot中对此也提供了自动化配置，可以简化静态资源过滤配置。

#### 默认策略
上文也提到，Spring Boot 中对于SpringMVC的自动化配置都在WebMvcAutoConfiguration类中，而在WebMvcAutoConfiguration类中有一个静态内部类WebMvcAutoConfigurationAdapter（适配器模式），实现了上面提到的WebMvcConfigurer接口。该接口中有一个addResourceHandlers方法，是用来配置静态资源过滤的。其具体实现是在WebMvcAutoConfigurationAdapter中，部分核心代码如下：

```java
public void addResourceHandlers (ResourceHandlerRegistry registry) {
    ...
        ...
    String staticPathPattern = this.mvcProperties.getStaticPathPattern();
        if (!registry.hasMappingForPattern(staticPathPattern)) {
            customizeResourceHandlerRegistration (
                registry.addResourceHandler(staticPathPattern)
                    .addResourceLocations (getResourceLocations (
                        this.resourceProperties.getStaticLocations ()))
                    .setCachePeriod(getSeconds(cachePeriod))
                    .setCacheControl(cacheControl));
        }
}    
```

Spring Boot 在这里进行了默认的静态资源过滤配置，其中staticPathPattern 默认定义在WebMvcProperties 中，定义内容如下：

```java
private String staticPathPattern = "/**";
```

this.resourceProperties.getStaticLocations（）获取到的默认静态资源位置定义在ResourceProperties中，代码如下：

```java
private static final String[] CLASSPATH_RESOURCE_LOCATIONS = (
"classpath:/META-INF/resources/", "classpath:/resources/", "classpath:/static/"，"classpath:/public/"};
```

在getResourceLocations 方法中， 对这4个静态资源位置做了扩充，代码如下：

```java
static String[] getResourceLocations (String[] staticLocations) {
String[] locations= new String[staticLocations.length + SERVLET_LOCATIONS.length];
System.arraycopy(staticLocations, 0, locations, 0, staticLocations.length);
System.arraycopy(SERVLET_LOCATIONS, 0, locations, staticLocations.length ,SERVLET LOCATIONS length);
return locations ;
```

其中， SERVLET_LOCATIONS 的定义是一个{"/"} 。

综上可以看到， Spring Boot 默认会过滤所有的静态资源， 而静态资源的位置一共有5个，分别是“classpath:/META-INF/resources/”、“classpath:/resources”、“classpath:/static”、“classpath:/public”以及“/”,也就是说，开发者可以将静态资源放到这5个位置中的任意一个。注意， 按照定义的顺序，5个静态资源位置的优先级依次降低。但是一般情况下， Spring Boot 项目不需要webapp 目录，所以第5 个“/”可以暂不考虑。

在一个新创建的Spring Boot 项目中，添加了spring-boot-starter-web 依赖之后，在resources 目录下分别创建4个目录，4个目录中放入同名的静态资源（如图4-4 所示， 数字表示不同位置资源的优先级）。

![资源目录]({{assets_base_url}}/images/blog/SpringBoot+Vue全栈开发实战/chapter03/资源目录.jpg)

此时，在浏览器中输入“http://localhost:8080/p1.png”即可看到classpath:/META-INF/resources/目录下的pl.png，如果将classpath:/META-INF/resources/目录下的pl.png 删除，就会访问到classpath:/resources/目录下的pl.png，以此类推。

如果使用Intellij IDEA创建Spring Boot 项目，则会默认创建出classpath:/static/ 目录，静态资源一般放在这个目录下即可。

#### 自定义策略
如果默认的静态资源过滤策略不能满足开发需求，也可以自定义静态资源过滤策略，静态资源过滤策略有以下两种方式：

1.在配置文件中定义

可以在application.peroperties中直接定义过滤规则和静态资源位置，代码如下：

```xml
spring.mvc.static-path-pattern=/static/**
spring.resources.static-location=classpath:/static/
```

如上，过滤规则为/static/**，静态资源位置为classpath:/static/。重新启动项目，在浏览器中输入“http://localhost:8080/static/p1.png”即可看到classpath:/static/目录下的资源。

2.Java编码定义

也可以通过Java编码方式定义，只需实现WebMvcConfigurer接口即可，然后实现该接口的addResourcHandlers方法，代码如下：

```java
@Configuration
public class MyWebMvcConfig implements WebMvcConfigurer {
  @Override
  public void addResourceHandlers(ResourceHandlerRegistry registry) {
    registry
            .addResourceHandler("/static/**")
            .addResourceLocations("classpath:/static/");
  }
}
```

### 文件上传
