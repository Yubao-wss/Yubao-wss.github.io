---
layout: post
title: 分布式Dubbo+Zookeeper+SpringBoot
categories:
  - ACM
  - Template
tags: Posts
date: 2020-10-14 14:00:00
---

### 分布式Dubbo+Zookeeper

#### 分布式系统的定义

- **分布式系统是若干独立计算机的集合，这些计算机对于用户来说就像是单个相关系统**
- 分布式系统是由一组通过网络进行通信、为了完成共同的任务而协调工作的计算机群组成的系统
- 分布式系统的出现是为了用廉价的、普通的机器完成单个计算机无法完成的计算、存储任务，其目的是利用更多的机器，处理更多的数据
- 只有当单个节点的处理能力无法满足要求，且硬件的提升得不偿失，程序也不能进一步优化的时候，我们才需要考虑分布式系统

#### Dubbo

服务架构模式发展历程如下图：

![dubbo-architecture-roadmap](https://img1.wenhairu.com/images/2020/10/14/CPmq3.jpg)

- 一个SpringBoot项目可以看做是一个MVC服务器，而RPC模式可以看作是多个SpringBoot项目的集合
- 这一群SpringBoot项目中可以分为两类：Service和API
- **其中RPC是一种进程间的通讯方式，用于程序间的远程调用，是一种技术思想，不是一种规范**
- **而Dubbo是一个基于java语言的开源RPC框架**
- RPC有两个核心技术：**通信和序列化**；Dubbo则包含了这些
- 除此之外Dubbo还包含了三大核心能力：**面向接口的远程方法调用、智能容错和负载均衡、服务自动注册和发现**

Dubbo框架下生产者与消费者工作流程图如下：

![architecture](https://img1.wenhairu.com/images/2020/10/14/CP8dK.png)

- 流程的顺序是生产者将自己注册在注册中心--->消费者订阅注册中心的服务--->注册中心通知消费者--->消费者调用生产者的服务--->服务调用完成，在监控中心进行记录
- 可以看到，而其中除了消费者调用生产者是同步的，其余的流程都是异步的

#### Zookeeper

Zookeeper就是上文所提到的注册中心的实现

- Zookeeper是 Apache Hadoop 的子项目，是一个树型的目录服务，支持变更推送，适合作为 Dubbo 服务的注册中心，工业强度较高，可用于生产环境 



#### 在SpringBoot项目中的使用

1. 安装并启动zookeeper

2. 下载dubbo-admin并打jar包（可选）,DubboAdmin\dubbo-admin\dubbo-admin路径下

   ```
   mvn clean package -Dmaven.test.skip=true
   ```

   一个dubbo监控管理后台，用于管理服务，默认端口是7001，默认密码root/root

3. 建立工程，创建provider模块

   - 增加依赖

     ```xml
     <dependency>
         <groupId>org.apache.dubbo</groupId>
         <artifactId>dubbo-spring-boot-starter</artifactId>
         <version>2.7.3</version>
     </dependency>
     
     <dependency>
         <groupId>com.github.sgroschupf</groupId>
         <artifactId>zkclient</artifactId>
         <version>0.1</version>
     </dependency>
     
     <dependency>
         <groupId>org.apache.curator</groupId>
         <artifactId>curator-framework</artifactId>
         <version>2.12.0</version>
     </dependency>
     <dependency>
         <groupId>org.apache.curator</groupId>
         <artifactId>curator-recipes</artifactId>
         <version>2.12.0</version>
     </dependency>
     <dependency>
         <groupId>org.apache.zookeeper</groupId>
         <artifactId>zookeeper</artifactId>
         <version>3.4.14</version>
     
         <exclusions>
             <exclusion>
                 <groupId>org.slf4j</groupId>
                 <artifactId>slf4j-log4j12</artifactId>
             </exclusion>
         </exclusions>
     </dependency>
     ```

   - 配置文件

     ```
     server.port=8001
     
     # 服务应用名字
     dubbo.application.name=provider-server
     # 注册中心地址
     dubbo.registry.address=zookeeper://127.0.0.1:2181
     # 哪些服务要被注册
     dubbo.scan.base-packages=cn.waston.service
     ```

   - 具体服务的接口

     ```java
     package cn.waston.service;
     
     public interface TicketService {
         public String getTicket();
     }
     ```

   - 具体服务的实现类

     ```java
     package cn.waston.service;
     
     import org.apache.dubbo.config.annotation.Service;
     import org.springframework.stereotype.Component;
     
     @Service //注意！ 这里是dubbo的@Service注解 加上后在项目启动时就可以被扫描到 自动注册到注册中心
     @Component
     public class TicketServiceImpl implements TicketService{
         @Override
         public String getTicket() {
             return "hello TicketServiceImpl";
         }
     }
     ```

   - 启动项目，在http://localhost:7001中就可以看到该服务的信息了

4. 创建consumer模块

   - 增加依赖，同上

   - 配置文件

     ```
     server.port=8002
     
     # 消费者暴露自己的名字
     dubbo.application.name=consumer-server
     # 注册中心的地址
     dubbo.registry.address=zookeeper://127.0.0.1:2181
     ```

   - 调用provider模块中服务的第一种方法

     - [ ] service包中添加一个与provider模块中相同的具体服务的接口，**包路径要完全一样**，如上述的TicketService

     - [ ] 创建自己的Service

       ```java
       package cn.waston.service;
       
       import org.apache.dubbo.config.annotation.Reference;
       import org.springframework.stereotype.Service;
       
       @Service//注意！ 这里是Spring的@Service
       public class UserService {
           @Reference// 引用 去注册中心拿到服务
           TicketService ticketService;
       
           public void buyTicket(){
               String ticket = ticketService.getTicket();
               System.out.println("在注册中心拿到：" + ticket);
           }
       
       }
       
       ```

     - [ ] 测试

       ```java
       @SpringBootTest
       class ConsumerServerApplicationTests {
       
           @Autowired
           UserService userService;
       
           @Test
           void contextLoads() {
               userService.buyTicket();
           }
       
       }
       ```

   - 第二种方法：pom文件配置消费者与生产者

     - [ ] provider模块添加相关依赖后，在resources包下增加一个providers.xml，内容如下

       ```xml
       <?xml version="1.0" encoding="UTF-8"?>
       <beans xmlns="http://www.springframework.org/schema/beans"
       	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:dubbo="http://code.alibabatech.com/schema/dubbo"
       	xsi:schemaLocation="http://www.springframework.org/schema/beans
              http://www.springframework.org/schema/beans/spring-beans.xsd
              http://code.alibabatech.com/schema/dubbo
              http://code.alibabatech.com/schema/dubbo/dubbo.xsd">
              
       	<!-- 配置可参考 http://dubbo.io/User+Guide-zh.htm -->
       	<!-- 服务提供方应用名，用于计算依赖关系 -->
       	<dubbo:application name="dubbo-provider" owner="dubbo-provider" />
       	
       	<!-- 定义 zookeeper 注册中心地址及协议 -->
       	<dubbo:registry protocol="zookeeper" address="127.0.0.1:2181" client="zkclient" />
       		
       	<!-- 定义 Dubbo 协议名称及使用的端口，dubbo 协议缺省端口为 20880，如果配置为 -1 或者没有配置 port，则会分配一个没有被占用的端口 -->
       	<dubbo:protocol name="dubbo" port="-1" />
       	
       </beans>
       ```

     - [ ] provider模块添加service接口与其实现类，这里的实现类可以不添加@Service注解

       ```java
       package cn.waston.service;
       
       public interface WastonService {
           public String getName();
       }
       ```

       ```java
       package cn.waston.service;
       
       public class WastonServiceImpl implements WastonService{
           @Override
           public String getName() {
               return "Sam";
           }
       }
       ```

     - [ ] resources包下增加一个providers-waston.xml

       ```xml
       <?xml version="1.0" encoding="UTF-8"?>
       <beans xmlns="http://www.springframework.org/schema/beans"
              xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:dubbo="http://code.alibabatech.com/schema/dubbo"
              xsi:schemaLocation="http://www.springframework.org/schema/beans
              http://www.springframework.org/schema/beans/spring-beans.xsd
              http://code.alibabatech.com/schema/dubbo
              http://code.alibabatech.com/schema/dubbo/dubbo.xsd">
       
           <!-- 声明需要暴露的服务接口 -->
           <dubbo:service interface="cn.waston.service.WastonService"
                          ref="WastonService" timeout="50000" />
           <bean id="WastonService" class="cn.waston.service.WastonServiceImpl" />
       
       </beans>
       ```

     - [ ] 启动类加注解，引入该配置文件

       ```java
       package cn.waston;
       
       import org.springframework.boot.SpringApplication;
       import org.springframework.boot.autoconfigure.SpringBootApplication;
       import org.springframework.context.annotation.ImportResource;
       
       @ImportResource(value={"classpath:providers-waston.xml"})
       @SpringBootApplication
       public class ProviderServerApplication {
       
           public static void main(String[] args) {
               SpringApplication.run(ProviderServerApplication.class, args);
           }
       
       }
       ```

     - [ ] consumer模块增加一个consumers.xml

       ```xml
       <?xml version="1.0" encoding="UTF-8"?>
       <beans xmlns="http://www.springframework.org/schema/beans"
              xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:dubbo="http://code.alibabatech.com/schema/dubbo"
              xsi:schemaLocation="http://www.springframework.org/schema/beans
              http://www.springframework.org/schema/beans/spring-beans.xsd
              http://code.alibabatech.com/schema/dubbo
              http://code.alibabatech.com/schema/dubbo/dubbo.xsd">
       
           <!-- 配置可参考 http://dubbo.io/User+Guide-zh.htm -->
           <!-- 消费方应用名，用于计算依赖关系，不是匹配条件，不要与提供方一样 -->
           <dubbo:application name="dubbo-consumer" owner="dubbo-consumer" />
       
           <!-- 定义 zookeeper 注册中心地址及协议 -->
           <dubbo:registry protocol="zookeeper" address="127.0.0.1:2181" client="zkclient" />
       </beans>
       ```

     - [ ] 创建于provider模块同路径、同内容的WastonService接口

     - [ ] classpath（resources包）下增加一个consumers-waston.xml

       ```xml
       <?xml version="1.0" encoding="UTF-8"?>
       <beans xmlns="http://www.springframework.org/schema/beans"
              xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:dubbo="http://code.alibabatech.com/schema/dubbo"
              xsi:schemaLocation="http://www.springframework.org/schema/beans
              http://www.springframework.org/schema/beans/spring-beans.xsd
              http://code.alibabatech.com/schema/dubbo
              http://code.alibabatech.com/schema/dubbo/dubbo.xsd">
       
           <!-- 生成远程服务代理，可以和本地 bean 一样使用 testService -->
           <dubbo:reference id="WastonService" interface="cn.waston.service.WastonService" />
       
       </beans>
       ```

     - [ ] 启动类引入该配置文件

       ```java
       package cn.waston;
       
       import org.springframework.boot.SpringApplication;
       import org.springframework.boot.autoconfigure.SpringBootApplication;
       import org.springframework.context.annotation.ImportResource;
       
       @ImportResource(value={"classpath:consumers-waston.xml"})
       @SpringBootApplication
       public class ConsumerServerApplication {
       
           public static void main(String[] args) {
               SpringApplication.run(ConsumerServerApplication.class, args);
           }
       
       }
       ```

     - [ ] 测试

       ```java
       @Reference// 引用 去注册中心拿到服务
       WastonService wastonService;
       
       @Test
       void test1(){
           System.out.println(wastonService.getName());
       }
       ```

   