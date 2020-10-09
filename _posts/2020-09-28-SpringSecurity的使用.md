---
layout: post
title: SpringSecurity的使用
categories:
  - ACM
  - Template
tags: Posts
date: 2020-09-28 14:00:00
---

### SpringSecurity简述

- Web开发中，安全十分重要，我们可以用过滤器、拦截器等方式实现一些安全保障
- 而SpringSecurity使用起来则更为方便
- SpringSecurity的同类框架还有Shiro
- SpringSecurity实现安全的主要方式：**认证、授权**

#### 哪些权限

- 功能权限
- 访问权限
- 菜单权限

### SpringSecurity的使用

1. 引入spring-starter-security模块

   ```xml
   <dependency>
       <groupId>org.springframework.boot</groupId>
       <artifactId>spring-boot-starter-security</artifactId>
   </dependency>
   ```

   记住下面几个类

   - WebSecurityConfigurerAdapter：自定义Security策略
   - AuthenticationManagerBuilder：自定义认证策略
   - @EnableWebSecurity：开启WebSecurity模式

2. 在项目中自定义配置类继承WebSecurityConfigurerAdapter，并重写其中的方法

   ```java
   @EnableWebSecurity
   public class SecurityConfig extends WebSecurityConfigurerAdapter {
       //授权
       @Override
       protected void configure(HttpSecurity http) throws Exception {
           //首页所有人可以访问，功能页只有对应权限的人可以访问
           http.authorizeRequests().antMatchers("/").permitAll()
               .antMatchers("/level1/**").hasRole("vip1")
               .antMatchers("/level2/**").hasRole("vip2")
               .antMatchers("/level3/**").hasRole("vip3");
   
           //没有权限自动跳到登陆页面
           http.formLogin();
           
           //开启注销
           http.logout();
       }
   
       //认证
       @Override
       protected void configure(AuthenticationManagerBuilder auth) throws Exception {
           //从内存中读取认证，一般情况是从数据库里读
           auth.inMemoryAuthentication().passwordEncoder(new BCryptPasswordEncoder())
                   .withUser("waston").password(new BCryptPasswordEncoder().encode("123456")).roles("vip1","vip2")
                   .and()
                   .withUser("root").password(new BCryptPasswordEncoder().encode("123456")).roles("vip1","vip2","vip3")
                   .and()
                   .withUser("guest").password(new BCryptPasswordEncoder().encode("123456")).roles("vip1");
       }
   }
   ```

   项目中的controller如下

   ```java
   @Controller
   public class RouterController {
   
       @RequestMapping({"/","/index"})
       public String index(){
           return "index";
       }
   
       @RequestMapping("/login")
       public String toLogin(){
           return "views/login";
       }
   
       @RequestMapping("/level1/{id}")
       public String level1(@PathVariable("id") int id){
           return "views/level1/"+id;
       }
   
       @RequestMapping("/level2/{id}")
       public String level2(@PathVariable("id") int id){
           return "views/level2/"+id;
       }
   
       @RequestMapping("/level3/{id}")
       public String level3(@PathVariable("id") int id){
           return "views/level3/"+id;
       }
   }
   ```

3. 通过上面的两步，即实现了**访问权限**的控制

4. configure方法中加入

   ```
   http.logout();
   ```

   前端页面写好注销按钮

   ```html
   <!--注销-->
   <a class="item" th:href="@{/logout}">
       <i class="sign-out icon"></i> 注销
   </a>
   ```

   即开启**注销功能**

5. **功能权限控制**的实现

   引入thymeleaf、security整合包

   ```xml
   <dependency>
       <groupId>org.thymeleaf.extras</groupId>
       <artifactId>thymeleaf-extras-springsecurity4</artifactId>
       <version>3.0.2.RELEASE</version>
   </dependency>
   ```

   **SpringBoot版本最高不能超过2.0.9**

   修改前端页面

   ```html
   <!--如果未登录，显示登陆按钮-->
   <div sec:authorize="!isAuthenticated()">
        <a class="item" th:href="@{/login}">
             <i class="address card icon"></i> 登录
        </a>
   </div>
   
   <!--如果登陆，显示用户信息-->
   <div sec:authorize="isAuthenticated()">
        <a class="item">
            用户名：<span sec:authentication="name"></span>
            角色：<span sec:authentication="principal.getAuthorities()"></span>
        </a>
   </div>
   
   <!--如果登陆，显示注销按钮-->
   <div sec:authorize="isAuthenticated()">
        <a class="item" th:href="@{/logout}">
             <i class="sign-out icon"></i> 注销
        </a>
   </div>
   ```

   记得引入命名空间

   ```
   xmlns:sec="http://www.thymeleaf.org/thymeleaf-extras-springsecurity4"
   ```

   如果出现注销失败，404的情况，可能是由于spring默认开启了csrf功能，关闭它

   configure方法中加一行

   ```java
   http.csrf().disable();
   ```

6. **菜单权限**的控制

   前端页面添加sec:authorize="hasRole('角色名称')"，如下

   ```html
   <div class="column" sec:authorize="hasRole('vip1')">
       <div class="ui raised segment">
           <div class="ui">
               <div class="content">
                   <h5 class="content">Level 1</h5>
                   <hr>
                   <div><a th:href="@{/level1/1}"><i class="bullhorn icon"></i> Level-1-1</a></div>
                   <div><a th:href="@{/level1/2}"><i class="bullhorn icon"></i> Level-1-2</a></div>
                   <div><a th:href="@{/level1/3}"><i class="bullhorn icon"></i> Level-1-3</a></div>
               </div>
           </div>
       </div>
   </div>
   ```

   

7. 开启**记住我**功能

   configure方法中加入

   ```
   http.rememberMe();
   ```

   即可开启记住我的功能，该功能的原理基于cookie，默认保存两周

   

8. 自定义登陆页面

   configure方法中加入

   ```java
   http.formLogin().loginPage("/tologin").loginProcessingUrl("/login");
   ```

   前端index页面

   ```html
   <div sec:authorize="!isAuthenticated()">
       <a class="item" th:href="@{/tologin}">
           <i class="address card icon"></i> 登录
       </a>
   </div>
   ```

   login页面

   ```html
   <form th:action="@{/login}" method="post">
                               <div class="field">
                                   <label>Username</label>
                                   <div class="ui left icon input">
                                       <input type="text" placeholder="Username" name="username">
                                       <i class="user icon"></i>
                                   </div>
                               </div>
                               <div class="field">
                                   <label>Password</label>
                                   <div class="ui left icon input">
                                       <input type="password" name="password">
                                       <i class="lock icon"></i>
                                   </div>
                               </div>
                               <input type="submit" class="ui blue submit button"/>
                           </form>
   ```

   controller

   ```java
   @RequestMapping("/tologin")
   public String toLogin(){
       return "views/login";
   }
   ```

   loginPage("/tologin")方法使系统在认证权限时，走到我们自定义的页面，而loginProcessingUrl("/login")方法设定了权限认证的地址，这里要和login页面的action一致

   -  Security认证时默认接收参数为**username**和**password**，当然我们也可以通过下面的方式自定义参数

   ```java
   http.formLogin().loginPage("/tologin")
           .usernameParameter("user")
           .passwordParameter("pwd")
           .loginProcessingUrl("/login");
   ```

   