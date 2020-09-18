---
layout: post
title: SpringBootWeb开发
categories:
  - ACM
  - Template
tags: Posts
date: 2020-09-17 14:00:00
---

# springBoot Web开发

### 静态资源

WebMvcAutoConfiguration类中有这样一个方法

```java
public void addResourceHandlers(ResourceHandlerRegistry registry) {
    if (!this.resourceProperties.isAddMappings()) {
        logger.debug("Default resource handling disabled");
    } else {
        Duration cachePeriod = this.resourceProperties.getCache().getPeriod();
        CacheControl cacheControl = this.resourceProperties.getCache().getCachecontrol().toHttpCacheControl();
        if (!registry.hasMappingForPattern("/webjars/**")) {
            this.customizeResourceHandlerRegistration(registry.addResourceHandler(new String[]{"/webjars/**"}).addResourceLocations(new String[]{"classpath:/META-INF/resources/webjars/"}).setCachePeriod(this.getSeconds(cachePeriod)).setCacheControl(cacheControl));
        }

        String staticPathPattern = this.mvcProperties.getStaticPathPattern();
        if (!registry.hasMappingForPattern(staticPathPattern)) {
            this.customizeResourceHandlerRegistration(registry.addResourceHandler(new String[]{staticPathPattern}).addResourceLocations(WebMvcAutoConfiguration.getResourceLocations(this.resourceProperties.getStaticLocations())).setCachePeriod(this.getSeconds(cachePeriod)).setCacheControl(cacheControl));
        }

    }
}
```

#### 第一种方式

可以看到其中有一个**/webjars/****

我们可以在pom.xml中添加如下依赖

```
<dependency>
    <groupId>org.webjars</groupId>
    <artifactId>jquery</artifactId>
    <version>3.4.1</version>
</dependency>
```

导入后，可以在依赖目录中找到它的jar包，其中jquery的js文件所在的文件目录是：jquery-3.4.1.jar/META-INF/resources/webjars/jquery/3.4.1，这与上面的方法中的

```
classpath:/META-INF/resources/webjars/
```

一致，所以springboot通过这样的方式，将一些静态资源加入了项目

- 启动项目后，可以在浏览器中输入ipaddress:port**/webjars**/jquery/3.4.1/jquery.js查看这个js文件
- 类似的webjars可以在<https://www.webjars.org/> 中找到

#### 第二种方式

可以看到方法中有一个staticPathPattern，它通过WebMvcProperties类中的如下方法获得

```java
public String getStaticPathPattern() {
    return this.staticPathPattern;
}
```

其中this.staticPathPattern的值是“**/********”

后面有一个this.resourceProperties，ResourceProperties类中有

```java
private static final String[] CLASSPATH_RESOURCE_LOCATIONS = new String[]{"classpath:/META-INF/resources/", "classpath:/resources/", "classpath:/static/", "classpath:/public/"};
```

- classpath：指**resource**下

- 可以看到有四个目录，这四个目录下的静态资源都能被识别到
- 故我们在启动项目后，在浏览器中输入ipaddress:port/xxx.xx，便可以访问到该静态资源，上述四个文件夹里的内容都可以被**ipaddress:port/**的访问接收
- 优先级：classpath:/resources/ > classpath:/static/ > classpath:/public/

#### **第三种方式**

可以看到方法中最前面有

```java
if (!this.resourceProperties.isAddMappings()) {
        logger.debug("Default resource handling disabled");
    } 
```

我们一旦在配置文件中设置

```
spring.mvc.static-path-pattern=/xxx/**
```

前面的默认路径都会失效，会按我们自己设置的路径走

### 首页如何定制

WebMvcAutoConfiguration类中有

```java
@Bean
public WelcomePageHandlerMapping welcomePageHandlerMapping(ApplicationContext applicationContext, FormattingConversionService mvcConversionService, ResourceUrlProvider mvcResourceUrlProvider) {
    WelcomePageHandlerMapping welcomePageHandlerMapping = new WelcomePageHandlerMapping(new TemplateAvailabilityProviders(applicationContext), applicationContext, this.getWelcomePage(), this.mvcProperties.getStaticPathPattern());
    welcomePageHandlerMapping.setInterceptors(this.getInterceptors(mvcConversionService, mvcResourceUrlProvider));
    welcomePageHandlerMapping.setCorsConfigurations(this.getCorsConfigurations());
    return welcomePageHandlerMapping;
}

private Optional<Resource> getWelcomePage() {
    String[] locations = WebMvcAutoConfiguration.getResourceLocations(this.resourceProperties.getStaticLocations());
    return Arrays.stream(locations).map(this::getIndexHtml).filter(this::isReadable).findFirst();
}

private Resource getIndexHtml(String location) {
    return this.resourceLoader.getResource(location + "index.html");
}
```

里面出现了

```java
this.mvcProperties.getStaticPathPattern()
getResource(location + "index.html")
```

所以，如果在那四个路径下存在index.html，就会被**ipaddress:port/**的访问直解返回浏览器展示

- 在templates目录下的页面，只能通过controller来跳转！
- controller类中的方法的返回值一般为“xxx”，xxx：**静态资源名称**，也可以加后缀

### 模板引擎Thymeleaf

模板引擎的作用：

- 我们在后端组装一些数据data，写一个嵌入表达式（模板引擎语法）的html页面
- 模板引擎会把html中的表达式解析，并按相应逻辑将data进行页面填充，最终返回一个填好数据的静态页面

ThymeleafProperties类中有

```java
public static final String DEFAULT_PREFIX = "classpath:/templates/";
public static final String DEFAULT_SUFFIX = ".html";
```

所以，**我们的模板文件必须写在templates目录下，而且必须以html结尾**

**结论**：只要需要使用thymeleaf，只需要导入对应的依赖就可以了

### MVC配置原理

Spring Boot为Spring MVC提供了自动配置，可与大多数应用程序完美配合。

自动配置在Spring的默认值之上添加了以下功能：

- 包含`ContentNegotiatingViewResolver`和`BeanNameViewResolver`。
- 支持提供静态资源，包括对WebJars的支持（[在本文档的后面部分中有介绍](https://docs.spring.io/spring-boot/docs/2.3.3.RELEASE/reference/html/spring-boot-features.html#boot-features-spring-mvc-static-content)）。
- 自动注册`Converter`，`GenericConverter`和`Formatter`Bean类。
- 支持`HttpMessageConverters`（[在本文档后面介绍](https://docs.spring.io/spring-boot/docs/2.3.3.RELEASE/reference/html/spring-boot-features.html#boot-features-spring-mvc-message-converters)）。
- 自动注册`MessageCodesResolver`（[在本文档后面介绍](https://docs.spring.io/spring-boot/docs/2.3.3.RELEASE/reference/html/spring-boot-features.html#boot-features-spring-message-codes)）。
- 静态`index.html`支持。
- 定制`Favicon`支持（[在本文档后面部分中介绍](https://docs.spring.io/spring-boot/docs/2.3.3.RELEASE/reference/html/spring-boot-features.html#boot-features-spring-mvc-favicon)）。
- 自动使用`ConfigurableWebBindingInitializer`bean（[在本文档后面介绍](https://docs.spring.io/spring-boot/docs/2.3.3.RELEASE/reference/html/spring-boot-features.html#boot-features-spring-mvc-web-binding-initializer)）。

如果要保留这些Spring Boot MVC定制并进行更多的[MVC定制](https://docs.spring.io/spring/docs/5.2.8.RELEASE/spring-framework-reference/web.html#mvc)（拦截器，格式化程序，视图控制器和其他功能），则可以添加自己的有`@Configuration`的类，**类名为**`WebMvcConfigurer`但**不添加** `@EnableWebMvc`。

如果你想提供的定制情况`RequestMappingHandlerMapping`，`RequestMappingHandlerAdapter`或者`ExceptionHandlerExceptionResolver`，仍然保持弹簧引导MVC自定义，你可以声明类型的Bean`WebMvcRegistrations`，并用它来提供这些组件的定制实例。

如果你想利用Spring MVC中的完全控制，你可以添加自己的`@Configuration`注解为`@EnableWebMvc`，或者添加自己的`@Configuration`-annotated `DelegatingWebMvcConfiguration`中的Javadoc中所述`@EnableWebMvc`。

上面为spring官方文档！



- 扩展MVC：自己写一个类，加`@Configuration`类名设置为`WebMvcConfigurer`

- 再加一个`@EnableWebMvc`则会全面接管!

- ```java
  @Configuration
  public class MyMvcConfig implements WebMvcConfigurer {
      
      @Bean
      public ViewResolver myViewResolver(){
          return new MyViewResolver();
      }
  
      //自定义试图解析器
      public static class MyViewResolver implements ViewResolver{
          @Override
          public View resolveViewName(String s, Locale locale) throws Exception {
              return null;
          }
      }
  }
  ```



