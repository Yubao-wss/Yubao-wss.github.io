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
- SpringSecurity实现安全的主要方式：认证、授权

#### 权限

- 功能权限
- 访问权限
- 菜单权限

### 使用

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

2. 