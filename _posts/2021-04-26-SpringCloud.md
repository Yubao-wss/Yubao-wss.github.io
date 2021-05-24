---
layout: post
title: SpringCloud
categories:
  - ACM
  - Template
tags: Posts
date: 2021-04-26 14:00:00
---

## SpringCloud

- 用于实现“微服务”架构，以及解决其问题的技术生态

### 微服务

- 简而言之，即模块化开发方式，把之前all in one（一站式）的系统，垂直拆分为若干模块
- 以更好的利用服务器资源，以及更合理的组织开发活动，最终为用户提供更多的价值

### 微服务架构需要解决的问题

1. 多个服务负载均衡的问题
2. 服务之间通信的问题
3. 服务崩溃的解决方法

### 微服务常见架构方案

- Spring Cloud NetFlix
- Apache Dubbo Zookeeper
- Spring Cloud Alibaba

### 技术栈一览

| 服务条目            | 落地技术                           |
| ------------------- | ---------------------------------- |
| 服务开发            | SpringBoot、SpringMVC              |
| 服务配置与管理      | Netfix的Archaius、阿里的Diamond    |
| 服务注册与发现      | Eureka、Consul、Zookeeper          |
| 服务调用            | Rest、RPC、gRPC                    |
| 服务熔断器          | Hystrix、Envoy                     |
| 负载均衡            | Ribbon、Nginx                      |
| 服务接口调用        | Feign                              |
| 消息队列            | Kafka、RabbitMQ、ActiveMQ          |
| 服务配置中心管理    | SpringCloudConfig、Chef            |
| 服务路由（API网关） | Zuul                               |
| 服务监控            | Zabbix、Nagios、Metrics、Specataor |
| 全链路追踪          | Zipkin、Brave、Dapper              |
| 服务部署            | Docker、OpenStack、Kubernetes      |
| 数据流操作开发包    | SpringCloud Stream                 |
| 事件消息总线        | SpringCloud Bus                    |

### Spring Cloud NetFlix

- 一个一站式解决方案
- 使用api网关、zuul组件
- 服务之间使用**Http**通信
- 使用Eureka实现服务注册与发现
- 利用Hystrix实现熔断机制

#### 对比Dubbo

- Dubbo底层是使用Netty这样的NIO框架，是基于TCP协议传输的，配合以Hession序列化完成RPC通信；而SpringCloud是基于Http协议+Rest接口调用远程过程的通信，相对来说，Http请求会有更大的报文，占的带宽也会更多
- REST相比RPC更为灵活，服务提供方和调用方的依赖只依靠一纸契约，**不存在代码级别的强依赖**，这在强调快速演化的微服务环境下，显得更为合适

#### NetFlix项目架构图

![netflix架构图](F:\Github\Yubao-wss.github.io\public\image\netflix架构图.jpg)

### Eureka

#### 概述

- Eureka是Netflix的一个子模块，是一个基于REST的服务，用于定位服务；实现了云端中层服务的发现和故障转移
- 功能类似于【注册中心】，同级产品还有Zookeeper
- 其采用了C-S的架构设计，EurekaServer作为服务注册功能的服务器，即服务注册中心
- 微服务模块使用Eureka的客户端连接到EurekaServer并维持心跳连接，系统维护人员可以通过EurekaServer来监控系统中各个模块是否正常运行

#### 对比Zookeeper

- CAP理论指出，一个分布式系统不可能同时满足C（一致性）、A（可用性）、P（容错性）
- Zookeeper保证的是CP，而Eureka保证的是AP
- Zookeeper存在一种问题，当集群中master节点故障时，剩余节点会进行master节点的选举，在选举过程中，整个集群都是不可用的，就会导致整个系统的暂时瘫痪
- Eureka集群与Zookeeper不同，不存在master的概念，每个节点都是平等的，都可以进行服务的注册和查询，即只要有一个节点在，就可以保证可用性

### Ribbon

#### 概述

- Spring Cloud Ribbon是基于Netfilx Ribbon实现的一套**客户端负载均衡工具**
- 主要功能是提供客户端的软件负载均衡算法，将NetFilx的中间服务层服务连接在一起

#### 对比Nginx

- Nginx属于集中式的负载均衡，是在消费方和提供方之间独立存在的，负责把访问请求通过某种策略转发至服务的提供方
- Ribbon属于进程式的负载均衡，是将其负载逻辑**集成到消费方**，消费方从注册中心获知有哪些地址可用，然后自己再从这些地址中选出一个合适的服务器

### Feign

- 使编写Java Http客户端变得更容易
- 在Feign的实现下，我们只需要创建一个接口并使用注解的方式来配置它，即可完成对服务提供方的接口绑定
- Feign集成了Ribbon
- Feign使代码的可读性提高，但会使程序的效率变低！

### Hystrix

#### 概述

- 分布式系统中，多个微服务之间调用的时候，如果某个调用的响应时间过长或不可用，对其前面的服务就会占用越来越多的系统资源，进而引起系统崩溃，即“雪崩效应”
- Hystrix（断路器）是一个用于处理分布式系统的延迟和容错的开源库，它能够保证在一个依赖出问题的情况下，不会导致整个服务失败，以提高分布式系统的弹性
- Hystrix的原理：当某个服务单元发生故障后，通过断路器的故障监测，**向调用方返回一个服务预期的，可处理的备选响应**，而不是长时间的等待或抛出异常

#### 功能

- 服务熔断

  当扇出链路的某个微服务不可用时，会熔断该节点微服务的调用，快速返回错误的响应信息。Hystrix会监控微服务间调用的状况，当失败的调用到一定阈值，一般是5秒内20次调用失败就会启动熔断机制，熔断机制的注解是@HystrixCommand

- 服务降级

  当遇到流量洪流时，由于服务器资源有限，可能需要暂停（关闭）一些服务，来为接收洪流或重要性更高的服务腾出空间，而为了避免雪崩以及给在此时访问被关闭服务的客户更好的体验，需要提供【降级服务】。

- 

### SpringCloud——Demo

1. 建立一个Maven父工程

   pom.xml如下

   ```
   <?xml version="1.0" encoding="UTF-8"?>
   <project xmlns="http://maven.apache.org/POM/4.0.0"
            xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
            xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
       <modelVersion>4.0.0</modelVersion>
   
       <groupId>cn.waston</groupId>
       <artifactId>spring_cloud_test</artifactId>
       <version>1.0-SNAPSHOT</version>
       <modules>
           <module>spring_cloud_api</module>
           <module>spring_cloud_provider</module>
       </modules>
   
       <!--打包方式为 pom-->
       <packaging>pom</packaging>
   
       <properties>
           <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
           <maven.compiler.source>1.8</maven.compiler.source>
           <maven.compiler.target>1.8</maven.compiler.target>
           <junit.version>4.12</junit.version>
       </properties>
   
       <dependencyManagement>
           <dependencies>
               <!--springCloud的依赖-->
               <dependency>
                   <groupId>org.springframework.cloud</groupId>
                   <artifactId>spring-cloud-dependencies</artifactId>
                   <version>Greenwich.SR1</version>
                   <type>pom</type>
                   <scope>import</scope>
               </dependency>
               <!--springBoot的依赖-->
               <dependency>
                   <groupId>org.springframework.boot</groupId>
                   <artifactId>spring-boot-dependencies</artifactId>
                   <version>2.1.4.RELEASE</version>
                   <type>pom</type>
                   <scope>import</scope>
               </dependency>
               <!--数据库-->
               <dependency>
                   <groupId>mysql</groupId>
                   <artifactId>mysql-connector-java</artifactId>
                   <version>8.0.22</version>
               </dependency>
               <dependency>
                   <groupId>com.alibaba</groupId>
                   <artifactId>druid</artifactId>
                   <version>1.1.10</version>
               </dependency>
               <!--Mybatis-->
               <dependency>
                   <groupId>org.mybatis.spring.boot</groupId>
                   <artifactId>mybatis-spring-boot-starter</artifactId>
                   <version>1.1.1</version>
               </dependency>
               <!--junit-->
               <dependency>
                   <groupId>junit</groupId>
                   <artifactId>junit</artifactId>
                   <version>${junit.version}</version>
               </dependency>
               <!--lombok-->
               <dependency>
                   <groupId>org.projectlombok</groupId>
                   <artifactId>lombok</artifactId>
                   <version>1.16.10</version>
               </dependency>
               <!--log4j-->
               <dependency>
                   <groupId>log4j</groupId>
                   <artifactId>log4j</artifactId>
                   <version>1.2.17</version>
               </dependency>
           </dependencies>
       </dependencyManagement>
   
       <build>
           <plugins>
               <plugin>
                   <groupId>org.springframework.boot</groupId>
                   <artifactId>spring-boot-maven-plugin</artifactId>
               </plugin>
           </plugins>
       </build>
   </project>
   ```

2. 新建子模块spring_cloud_api

   pom:

   ```
   <?xml version="1.0" encoding="UTF-8"?>
   <project xmlns="http://maven.apache.org/POM/4.0.0"
            xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
            xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
       <parent>
           <artifactId>spring_cloud_test</artifactId>
           <groupId>cn.waston</groupId>
           <version>1.0-SNAPSHOT</version>
       </parent>
       <modelVersion>4.0.0</modelVersion>
   
       <artifactId>spring_cloud_api</artifactId>
   
       <!--当前Model自己需要的依赖 自动指向父工程-->
       <dependencies>
           <dependency>
               <groupId>org.projectlombok</groupId>
               <artifactId>lombok</artifactId>
           </dependency>
       </dependencies>
   </project>
   ```

   创建一个实体类

   ```java
   @Data
   @NoArgsConstructor
   @Accessors(chain = true) // 开启链式写法
   public class Dept implements Serializable {
       private Long deptno;
       private String dname;
       private String db_source;
   
       public Dept(String dname) {
           this.dname = dname;
       }
   }
   ```

3. 新建子模块spring_cloud_provider（此模块作为服务提供者）

   搭建【三层架构】，其中会用到上面的Dept

   ```java
   @RestController
   public class DeptController {
       @Autowired
       private DeptService deptService;
   
       @PostMapping("/dept/add")
       public boolean addDept(Dept dept){
           return deptService.addDept(dept);
       }
   
       @GetMapping("/dept/get/{id}")
       public Dept get(@PathVariable("id") Long id){
           return deptService.queryById(id);
       }
   
       @GetMapping("/dept/list")
       public List<Dept> queryAll(){
           return deptService.queryAll();
       }
   }
   ```

4. 新建子模块spring_cloud_consumer（此模块作为服务消费者）

   此模块只创建一个Controller

   ```java
   @RestController
   public class DeptConsumerController {
       @Autowired
       private RestTemplate restTemplate;
   
       private static final String REST_URL_PREFIX = "http://localhost:8001";
   
       @RequestMapping("/consumer/dept/add")
       public boolean add(Dept dept){
           return restTemplate.postForObject(REST_URL_PREFIX + "/dept/add", dept, Boolean.class);
       }
   
       @RequestMapping("/consumer/dept/get/{id}")
       public Dept get(@PathVariable("id") Long id){
           return restTemplate.getForObject(REST_URL_PREFIX + "/dept/get/" + id, Dept.class);
       }
   
       @RequestMapping("/consumer/dept/list")
       public List<Dept> list(){
           return restTemplate.getForObject(REST_URL_PREFIX + "/dept/list", List.class);
       }
   }
   ```

   不创建以及引入service代码，只是通过【restTemplate】来调用生产者的接口

   restTemplate则由自己注入

   ```java
   @Configuration
   public class ConfigBean {
       @Bean
       public RestTemplate getRestTemplate(){
           return new RestTemplate();
       }
   }
   ```

5. 建立模块spring_cloud_eureka（作为注册中心）

   添加依赖

   ```
   <dependencies>
       <dependency>
           <groupId>org.springframework.cloud</groupId>
           <artifactId>spring-cloud-starter-eureka-server</artifactId>
           <version>1.4.6.RELEASE</version>
       </dependency>
   </dependencies>
   ```

   配置Eureka

   ```
   server:
     port: 7777
   
   #Eureka配置
   eureka:
     instance:
       hostname: localhost
     client:
       register-with-eureka: false #是否向注册中心注册自己
       fetch-registry: false #表示自己为注册中心
       service-url: #监控页面的地址
         defaultZone: http://${eureka.instance.hostname}:${server.port}/eureka/
   ```

   启动类

   ```java
   @SpringBootApplication
   @EnableEurekaServer
   public class EurekaServer {
       public static void main(String[] args) {
           SpringApplication.run(EurekaServer.class,args);
       }
   }
   ```

6. 将spring_cloud_provider模块注册到Eureka

   添加依赖

   ```
   <!--依赖api模块 使用其中的实体类-->
   <dependency>
       <groupId>cn.waston</groupId>
       <artifactId>spring_cloud_api</artifactId>
       <version>1.0-SNAPSHOT</version>
   </dependency>
   <!--完善监控信息-->
   <dependency>
       <groupId>org.springframework.boot</groupId>
       <artifactId>spring-boot-starter-actuator</artifactId>
   </dependency>
   ```

   添加配置

   ```
   eureka:
     client:
       service-url:
         defaultZone: http://localhost:7777/eureka/
     instance:
       instance-id: spring_cloud_provider # 描述信息 对应Eureka监控页面的Status
   
   # 服务的详细信息 在Eureka监控页面可以查看
   info:
     app.name: test_springcloud
     company.name: cn.waston
   ```

   启动类添加一个注释

   ```
   @EnableEurekaClient
   ```

   重启该服务，将其注册到Eureka注册中心

7. 把spring_cloud_provider模块复制几份，分别启动，实现服务【提供者集群】

   注意：他们配置的spring.application.name需要完全一致，而eureka.instance.instance-id应该不同，以便在监控页面易于区分

8. spring_cloud_consumer模块中集成Ribbon以实现负载均衡

   引入依赖

   ```
   <!--Ribbon-->
   <dependency>
       <groupId>org.springframework.cloud</groupId>
       <artifactId>spring-cloud-starter-ribbon</artifactId>
       <version>1.4.6.RELEASE</version>
   </dependency>
   <!--Eureka-->
   <dependency>
       <groupId>org.springframework.cloud</groupId>
       <artifactId>spring-cloud-starter-eureka</artifactId>
       <version>1.4.6.RELEASE</version>
   </dependency>
   ```

   添加配置

   ```
   eureka:
     client:
       register-with-eureka: false #不向Eureka注册自己
       service-url:
         defaultZone: http://localhost:7777/eureka/
   ```

   启动类添加@EnableEurekaClient

   实现负载均衡，只需加一个注解

   ```java
   @Configuration
   public class ConfigBean {
       @Bean
       @LoadBalanced //配置负载均衡实现
       public RestTemplate getRestTemplate(){
           return new RestTemplate();
       }
   }
   ```

   ps：@LoadBalanced原理：<https://fangshixiang.blog.csdn.net/article/details/100788040> 

   Controller中原先的地址修改为provider模块配置的服务名

   ```
   private static final String REST_URL_PREFIX = "http://SPRINGCLOUD-PROVIDER-DEPT";
   ```

   ps:自定义Ribbon算法

   启动类上加注释，如：

   ```
   @RibbonClient(name = "SPRINGCLOUD-PROVIDER-DEPT", configuration = MyRule.class)
   ```

   name的值为服务名

   官方在这里给出警告：MyRule类不能和启动类在同级或子集目录下，否则会被@ComponentScan扫描到，然后被所有@RibbonClient共享

   ```java
   @Configuration
   public class MyRule {
       @Bean
       //必须为IRule的实现类
       public IRule myRule2(){
           return new RoundRobinRule();
       }
   }
   ```

9. Feign的使用

   于spring_cloud_api模块添加Feign依赖

   ```
   <dependency>
       <groupId>org.springframework.cloud</groupId>
       <artifactId>spring-cloud-starter-feign</artifactId>
       <version>1.4.6.RELEASE</version>
   </dependency>
   ```

   创建一个DeptClientService

   ```java
   @Component
   @FeignClient(value = "SPRINGCLOUD-PROVIDER-DEPT")
   public interface DeptClientService {
       @GetMapping("/dept/get/{id}")
       public Dept queryById(@PathVariable("id") Long id);
   
       @GetMapping("/dept/list")
       public List<Dept> queryAll();
   
       @PostMapping("/dept/add")
       public boolean addDept(Dept dept);
   }
   ```

   之前的Consumer模块的Controller则可以写为如下形式

   ```java
   @RestController
   public class DeptConsumerController {
       @Autowired
       private RestTemplate restTemplate;
   
       @Autowired
       private DeptClientService deptClientService;
   
       @RequestMapping("/consumer/dept/add")
       public boolean add(Dept dept){
           return deptClientService.addDept(dept);
       }
   
       @RequestMapping("/consumer/dept/get/{id}")
       public Dept get(@PathVariable("id") Long id){
           return deptClientService.queryById(id);
       }
   
       @RequestMapping("/consumer/dept/list")
       public List<Dept> list(){
           return deptClientService.queryAll();
       }
   }
   ```

   Consumer模块启动类加上

   ```
   @EnableFeignClients(basePackages = {"cn.waston.springcloud"})
   ```

   PS:DeptClientService中的接口必须与JProvider模块中的完全一致！

10. Hystrix的使用

   断路器：

   spring_cloud_provider添加依赖

   ```
   <dependency>
       <groupId>org.springframework.cloud</groupId>
       <artifactId>spring-cloud-starter-hystrix</artifactId>
       <version>1.4.6.RELEASE</version>
   </dependency>
   ```

   重构接口

   ```java
   @RestController
   public class DeptController {
       @Autowired
       private DeptService deptService;
   
       @Autowired
       private DiscoveryClient discoveryClient;
   
       @GetMapping("/dept/get/{id}")
       @HystrixCommand(fallbackMethod = "hystrixGet")
       public Dept get(@PathVariable("id") Long id){
           Dept dept = deptService.queryById(id);
   
           if(dept == null){
               throw new RuntimeException("id:" + id + " 不存在");
           }
   
           return dept;
       }
   
       public Dept hystrixGet(@PathVariable("id") Long id){
           return new Dept().setDeptno(id).setDname("id:" + id + " 不存在");
       }
   
   }
   ```

   启动类添加注解

   ```
   @EnableCircuitBreaker
   ```

   

   服务降级：

   spring_cloud_api模块，增加降级服务回调类

   ```java
   @Component
   public class DeptClientServiceFallbackFactory implements FallbackFactory {
       @Override
       public DeptClientService create(Throwable throwable) {
           return new DeptClientService() {
               @Override
               public Dept queryById(Long id) {
                   return new Dept()
                           .setDeptno(id)
                           .setDname("没有对应信息，服务降级");
               }
   
               @Override
               public List<Dept> queryAll() {
                   return null;
               }
   
               @Override
               public boolean addDept(Dept dept) {
                   return false;
               }
           };
       }
   }
   ```

   DeptClientService接口增加注解信息

   ```
   @FeignClient(value = "SPRINGCLOUD-PROVIDER-DEPT",fallbackFactory = DeptClientServiceFallbackFactory.class)
   ```

   spring_cloud_feign模块增加有关Hystrix的配置

   ```
   feign.hystrix.enabled=true
   ```

   启动相关模块，在spring_cloud_feign模块被关闭时，客户端就可以接收到降级的服务

   

   Hystrix监控页面

   新建模块spring_cloud_consumer_hystrix_dashboard

   依赖同spring_cloud_consumer模块，再添加

   ```
   <dependency>
       <groupId>org.springframework.cloud</groupId>
       <artifactId>spring-cloud-starter-hystrix</artifactId>
       <version>1.4.6.RELEASE</version>
   </dependency>
   <dependency>
       <groupId>org.springframework.cloud</groupId>
       <artifactId>spring-cloud-starter-hystrix-dashboard</artifactId>
       <version>1.4.6.RELEASE</version>
   </dependency>
   ```

   启动类添加

   ```
   @EnableHystrixDashboard
   ```

   Provider模块注入一个Bean

   ```java
   @Bean
   public ServletRegistrationBean hystrixMetricsStreamServlet(){
       ServletRegistrationBean registrationBean = new ServletRegistrationBean(new HystrixMetricsStreamServlet());
       registrationBean.addUrlMappings("/actuator/hystrix.stream");
       return registrationBean;
   }
   ```

   启动相关模块

   访问：http://xxxx:xxx/hystrix，可以看到查询页面，中间输入框输入  http://xxxx:xxx/actuator/hystrix.stream，查看具体服务的监控信息

   

11. 