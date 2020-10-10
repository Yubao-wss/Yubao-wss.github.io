---
layout: post
title: Swagger的使用
categories:
  - ACM
  - Template
tags: Posts
date: 2020-09-29 14:00:00
---

## Swagger

#### 作用和概念

##### 前后端分离

- 目前使用最多的前后端分离架构技术框架组合：Vue + SpringBoot

- 后端时代：前端只用管理静态页面，后端再将静态页面改为模板引擎（JSP）

- 前后端分离时代：

  - 后端：后端控制层、服务层、数据访问层

  - 前端：前端控制层、视图层

- 前后端交互：API

- 前后端相对独立：松耦合

- 前后端可以部署在不同的服务器上

- 前后端服务都可以启动多个，适应高并发场景

##### 前后端分离产生的问题

- 前后端集成联调，前端人员和后端人员无法做到**及时协商**
- 解决方案（早期）：
  - 制定一个schema（计划），实时更新API
  - word文档记录API，按时更新
  - 前端用postman测试
- 当下流行解决方案：**Swagger**

#### 特点

- API文档自动生成（API文档与API定义同步更新）
- 可以在线测试API接口
- 支持多种语言

#### 在项目中使用Swagger

1. 导入相关依赖

   ```xml
   <!-- https://mvnrepository.com/artifact/io.springfox/springfox-swagger2 -->
   <dependency>
       <groupId>io.springfox</groupId>
       <artifactId>springfox-swagger2</artifactId>
       <version>2.9.2</version>
   </dependency>
   <!-- https://mvnrepository.com/artifact/io.springfox/springfox-swagger-ui -->
   <dependency>
       <groupId>io.springfox</groupId>
       <artifactId>springfox-swagger-ui</artifactId>
       <version>2.9.2</version>
   </dependency>
   
   ```

   

2. 增加Swagger配置类

   ```java
   @Configuration
   @EnableSwagger2
   public class SwaggerConfig {
   }
   ```

3. 这时，访问**localhost:8080/swagger-ui.html**，即可看到Swagger的图形化界面

#### 配置Swagger

1. 配置类中加入docket实例

   ```java
   @Configuration
   @EnableSwagger2
   public class SwaggerConfig {
       @Bean
       public Docket docket(){
           return new Docket(DocumentationType.SWAGGER_2);
       }
   
   }
   ```

   

2. 继续加入一个apiInfo用来自定义swagger信息

   ```java
   @Configuration
   @EnableSwagger2
   public class SwaggerConfig {
       @Bean
       public Docket docket(){
           return new Docket(DocumentationType.SWAGGER_2)
                   .apiInfo(apiInfo());
       }
   
       private ApiInfo apiInfo(){
   
           //作者信息
           Contact contact = new Contact("Waston","https://yubao-wss.github.io/","xxx@xx.com");
   
           return new ApiInfo(
                   "测试Swagger",
                   "专门用于测试的项目",
                   "1.0",
                   "https://yubao-wss.github.io/",
                   contact,
                   "Apache 2.0",
                   "http://www.apache.org/licenses/LICENSE-2.0",
                   new ArrayList()
           );
       }
   }
   ```
   

#### Swagger配置扫描接口

设置扫描内容后，Swagger页面上就只显示指定扫描的内容，**注意前有select()后有build()**

1. 指定要扫描的包apis()

   ```java
   @Bean
   public Docket docket(){
       return new Docket(DocumentationType.SWAGGER_2)
               .apiInfo(apiInfo())
               .select()
               //RequestHandlerSelectors:配置要扫描接口的方式
               .apis(RequestHandlerSelectors.basePackage("cn.waston.testswagger.controller"))
               .build();
   }
   ```

2. 其他扫描方式

   ```java
   //扫描全部
   .apis(RequestHandlerSelectors.any())
   //不扫描
   .apis(RequestHandlerSelectors.none())
   //扫描有指定注解的类下的内容
   .apis(RequestHandlerSelectors.withClassAnnotation(RestController.class))
   //扫描有指定注解的方法
   .apis(RequestHandlerSelectors.withMethodAnnotation(GetMapping.class)
   //
   
   ```

   

3. 过滤path()，如下这样就过滤掉了指定路径下的内容

   ```java
   @Bean
   public Docket docket(){
       return new Docket(DocumentationType.SWAGGER_2)
               .apiInfo(apiInfo())
               .paths(PathSelectors.ant("/waston/**"))
               .build();
   }
   ```

#### 配置是否启动Swagger

enable控制是否启动Swagger，如果设置为false，swagger就不会启动了

```java
@Bean
public Docket docket(){
    return new Docket(DocumentationType.SWAGGER_2)
            .apiInfo(apiInfo())
            .enable(false)
}
```

#### 生产环境使用Swagger，而正式环境不使用

1. 启用多配置

   创建application-dev.properties和application-pro.properties

   在application.properties文件中指定使用的配置文件，如下

   ```
   spring.profiles.active=dev
   ```

   

2. Swagger配置类中，如下

   ```java
   @Bean
   public Docket docket(Environment environment){
       
       //设置要显示的Swagger环境
       Profiles profiles = Profiles.of("dev");
       
       //判断是否处在自己设定的环境
       boolean b = environment.acceptsProfiles(profiles);
   
       return new Docket(DocumentationType.SWAGGER_2)
               .apiInfo(apiInfo())
               .enable(b)
   }
   ```

   

#### 配置API文档的分组

分组功能一般用于协同开发，分组功能的实现如下

```java
@Configuration
@EnableSwagger2
public class SwaggerConfig {
    
    @Bean
    public Docket docket1(){
        return new Docket(DocumentationType.SWAGGER_2).groupName("A");
    }

    @Bean
    public Docket docket2(){
        return new Docket(DocumentationType.SWAGGER_2).groupName("B");
    }

    @Bean
    public Docket docket3(){
        return new Docket(DocumentationType.SWAGGER_2).groupName("C");
    }
}
```

配置类中定义多个不同的bean并通过groupName方法指定组名即可



#### 扫描实体类

我们在Controller类中所定义的接口里所返回的所有实体类，都会被扫描，并在SwaggerAPI页面显示

```java
@GetMapping("/user")
public User user(){
    return new User();
}
```

可以使用如下的方法为实体类增加注释，以便在Swagger界面查看

```java
@ApiModel("用户实体类")
public class User {
    @ApiModelProperty("用户名")
    public String username;
    @ApiModelProperty("密码")
    public String password;
}
```

#### 接口注释

使用@ApiOperation与@ApiParam可以分别为接口的方法与参数添加注释

```java
@RestController
public class HelloController {

    @ApiOperation("hello方法")
    @GetMapping(value = "/hello")
    public String hello(@ApiParam("用户名") String username){
        return "hello" + username;
    }

    @GetMapping("/user")
    public User user(){
        return new User();
    }
}
```