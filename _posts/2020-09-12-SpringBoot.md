---
layout: post
title: SpringBoot原理
categories:
  - ACM
  - Template
tags: Posts
date: 2020-09-16 14:46:00
---

# spring

为了降低Java的开发复杂性，Spring采用了以下4种关键策略

1、基于POJO（Bean）的轻量级和最小入侵编程

2、通过IOC，依赖注入和面向接口实现松耦合

3、基于切面（AOP）和惯例进行声明式编程

4、通过切面和模板减少样式代码

## Spingboot

是一个javaweb的**开发框架**，和SpringMVC类似，对比其他javaweb框架的优势是**约定大于配置**。

不是新的框架，默认配置了很多框架的使用方式，就像是maven整合了所有的jar包，spring boot整合了所有的框架



## 微服务

一种架构风格，把一个应用构建成一系列小服务（模块）的组合；可以通过http（RPC）的方式进行互通

打破之前all in one的架构方式，把每个功能元素独立出来，把独立的功能元素进行动态组合，所以微服务架构是对**功能元素进行复制，而没有对整个应用进行复制**

​	好处是

1、节省了调用资源

2、每个功能元素的服务都是一个可替换的、可独立升级



## Spingboot原理

#### 自动配置：

pom.xml

-  spring-boot-dependencies:核心依赖在父工程中！

- 我们在写或者引入一些SPringboot依赖的时候，不需要指定版本（web、test等），因为父依赖指定了版本

  启动器

- 启动器spring-boot-starter就是Springboot的启动场景

- 比如spring-boot-starter-web，加上它就会自动导入web环境所有的依赖！

- springboot会将所有的功能场景，都变成一个个的启动器

- 我们需要什么功能，只需要找到对应的启动器就可以了（Spring官网上有）​

注解（一个tab表示一层继承）

```java
@SpringBootApplication：标注这是一个springboot的应用;通过以下的注解导入所有定义的配置
	@SpringBootConfiguration（）：springboot的配置
		@Configuration：spring配置类
			@Component：说明这也是一个spring的组件
	@EnableAutoConfiguration：启用自动配置
		@AutoConfigurationPackage：自动配置
			@Import({Registrar.class})：包注册
		@Import({AutoConfigurationImportSelector.class})：自动导入选择器

```

@Import通过快速导入的方式实现把实例加入spring的IOC容器中 ；**@Import注解可以用于导入第三方包** 

META-INF/spring.factories（org.springframework.boot:springboot:autoconfigure包中）：自动配置的核心文件；所有的自动配置都在这里面

**结论**：springboot所有自动配置都是在启动的时候扫描并加载（扫描上面的spring.factories），但不一定全部生效，要判断条件是否成立，只要导入了对应的start，就有对应的启动器了，有了启动器，自动配置就会生效，然后配置生效！

1、springboot在启动的时候，从类路径下META-INF/spring.factories获取指定的值

2、将这些自动配置的类导入容器，自动配置就会生效，帮我们进行自动配置

3、以前需要我们自己配置的东西，现在springboot帮我们做了

4、整个javaEE解决方案和自动配置的东西都在spring-boot-autoconfigure-2.2.0.RELEASE.jar这个包下

5、它会把所有需要导入的组件，以类名的方式返回，这些组件就会被添加到容器

6、容器中也会存在非常多的xxxAutoConfiguration文件（都有@Configuration注解）（其中有很多@Bean），就是这些类给容器中导入	了这个场景需要的所有组件，并自动配置

7、有了自动配置类，免去了我们手动编写配置文件的工作

 **自定义配置**：

1、application.properties/application.yaml文件中去写，可以修改SpringBoot自动配置的默认值

- yaml文件还可以给实体类赋值：实体类上加@ConfigurationProperties(prefix = "yaml中的对象名")
- properties文件给实体类属性赋值：类上加@PropertySource(value = "classpath:xxx.properties")，属性值上加@Value("${properties里的属性名}")

2、自定义有@Configuration注解的类

```java
@Configuration
public class WebMVCConfig implements WebMvcConfigurer {
    @Override
    public void addResourceHandlers(ResourceHandlerRegistry registry) {
        registry.addResourceHandler("/**").addResourceLocations("classpath:/aaa/");
    }
}
```



#### 启动类

```java
@SpringBootApplication
public class XXXXApplication {
    public static void main(String[] args) {
        SpringApplication.run(XXXXApplication.class, args);
    }
}
```

SpringApplication类（run方法）做了四件事情

1、推断应用的类型是普通的项目还是Web项目

2、查找并加载所有可用初始化器，设置到initializers属性中

3、找出所有的应用程序监听器，设置到listeners属性中

4、推断并设置main方法的定义类，找到运行的主类

























































