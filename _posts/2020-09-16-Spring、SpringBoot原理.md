---
layout: post
title: Spring、SpringBoot原理
categories:
  - ACM
  - Template
tags: Posts
date: 2020-09-16 14:46:00
---

## spring

### Spring简介

**Spring**是一个轻量级的控制反转（IoC）和面向切面（AOP）的容器框架，其出现是为了降低软件开发的复杂性

为了降低Java的开发复杂性，Spring主要采用了以下4种关键策略

1. 基于POJO（Bean）的轻量级和最小入侵编程
2. 通过IoC，依赖注入和面向接口实现松耦合
3. 基于切面（AOP）和惯例进行声明式编程
4. 通过切面和模板减少样式代码

### Spring原理

#### 控制反转（IoC）

##### IoC的作用及实现

在Spring之前的传统Java开发中，业界通常用如：Controller->Service->Dao这样的分层方式来进行代码的组织，Service层会直接创建Dao层的对象来进行工作，即**组件的调用方参与了组件的创建和配置工作**，进而形成了耦合，导致代码在改动时的工作量很大；同时，下层类可能被创建了多个相同的对象，致使我们只做到了**逻辑复用**，并没有做到**资源复用** ，在这样架构下，每个类的对象都应该是单例的才对！

IoC就是为了解决上述的问题而诞生，它通过控制反转的方式，降低了代码的耦合，同时避免了资源的浪费

**控制反转**在Spring中的实现为：对象通过【配置文件】创建并交由IoC容器管理，交由容器管理后的对象称之为 Bean。调用方不再负责组件（对象）的创建，要使用组件时直接获取对应Bean 即可（通过类属性+注解，实质是调用set方法）。而这种从容器获取Bean的方式被称为**依赖注入** 

参考文章：<https://zhuanlan.zhihu.com/p/355778555> 

程序实例：

通过配置文件注入对象

```java
public interface UserDao {
    void getUser();
}

public class HighUserDaoImpl implements UserDao {
    public void getUser() {
        System.out.println("高级用户数据");
    }
}

public class LowUserDaoImpl implements UserDao{
    public void getUser() {
        System.out.println("低级用户数据");
    }
}

public interface UserService {
    void getUser();
}

public class UserServiceImpl implements UserService{

    private UserDao userDao;

    public void getUser() {
        userDao.getUser();
    }

    public UserDao getUserDao() {
        return userDao;
    }

    public void setUserDao(UserDao userDao) {
        this.userDao = userDao;
    }
}

//测试
public class MyTest {

    public static void main(String[] args) {
        ClassPathXmlApplicationContext context = new ClassPathXmlApplicationContext("beans.xml");
        UserServiceImpl userService = (UserServiceImpl) context.getBean("userService");
        userService.getUser();
    }
}
```

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:p="http://www.springframework.org/schema/p"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="highUserDao" class="cn.waston.dao.HighUserDaoImpl"></bean>

    <bean id="lowUserDao" class="cn.waston.dao.LowUserDaoImpl"></bean>

    <bean id="userService" class="cn.waston.service.UserServiceImpl">
        <property name="userDao" ref="highUserDao"></property>
    </bean>
</beans>
```

- 对象是如何创建（注入到IoC容器）的？

有三种主流方式：

1. Spring框架会解析我们写好的配置文件，调用无参构造方法创建对象、调用set方法给属性赋值

   ```
   <bean id="userService" class="cn.waston.service.UserServiceImpl">
           <property name="userDao" ref="highUserDao"></property>
   </bean>
   ```

2. 调用有参构造，下标赋值

   ```
   <bean id="userService2" class="cn.waston.service.UserServiceImpl">
       <constructor-arg index="0" ref="lowUserDao"></constructor-arg>
   </bean>
   ```

3. 调用有参构造，类型赋值（构造方法中不能有相同类型）

   ```
   <bean id="userService" class="cn.waston.service.UserServiceImpl">
       <constructor-arg type="cn.waston.dao.UserDao" ref="lowUserDao"></constructor-arg>
   </bean>
   ```

此外，还有p命名空间注入，c命名空间注入等扩展方式

##### Bean的自动装配

Spring会在上下文中自动寻找并给bean装配属性

Spring中有三种自动装配的方式

1. 在xml文件中显式的配置

   ```xml
   <?xml version="1.0" encoding="UTF-8"?>
   <beans xmlns="http://www.springframework.org/schema/beans"
          xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
          xmlns:p="http://www.springframework.org/schema/p"
          xsi:schemaLocation="http://www.springframework.org/schema/beans
           http://www.springframework.org/schema/beans/spring-beans.xsd">
   
       <bean id="cat" class="cn.waston.pojo.Cat"></bean>
   
       <bean id="dog" class="cn.waston.pojo.Dog"></bean>
   
       <!--
       byName：自动在容器上下文中查找和类中setXxx方法中与xxx相同的beanId的bean，并调用这个set方法进行装配
       byType：自动在容器上下文中查找和类中属性相同的bean，并调用对应的set方法装配 ps：上下文中该类的bean必须唯一
       -->
       <bean id="person" class="cn.waston.pojo.Person" autowire="byName">
           <property name="name" value="sam"></property>
       </bean>
   </beans>
   ```

   ```java
   public class Cat {
       
   }
   
   public class Dog {
    
   }
   
   public class Person {
   
       private Cat cat;
       private Dog dog;
       private String name;
   
       public Cat getCat() {
           return cat;
       }
   
       public void setCat(Cat cat) {
           this.cat = cat;
       }
   
       public Dog getDog() {
           return dog;
       }
   
       public void setDog(Dog dog) {
           this.dog = dog;
       }
   
       public String getName() {
           return name;
       }
   
       public void setName(String name) {
           this.name = name;
       }
   
       @Override
       public String toString() {
           return "Person{" +
                   "cat=" + cat +
                   ", dog=" + dog +
                   ", name='" + name + '\'' +
                   '}';
       }
   }
   
   public class TestAutowire {
   
       @Test
       public void testAutowire(){
           ApplicationContext context = new ClassPathXmlApplicationContext("beans.xml");
           Person person = (Person) context.getBean("person");
           System.out.println(person.toString());
       }
   }
   ```

2. 在Java代码中显式配置（注解式）

   xml中开启对注解式配置的支持

   ```xml
   <?xml version="1.0" encoding="UTF-8"?>
   <beans xmlns="http://www.springframework.org/schema/beans"
          xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
          xmlns:context="http://www.springframework.org/schema/context"
          xmlns:p="http://www.springframework.org/schema/p"
          xsi:schemaLocation="http://www.springframework.org/schema/beans
           http://www.springframework.org/schema/beans/spring-beans.xsd
           http://www.springframework.org/schema/context
           http://www.springframework.org/schema/context/spring-context.xsd">
   
       <bean id="cat" class="cn.waston.pojo.Cat"/>
   
       <bean id="dog" class="cn.waston.pojo.Dog"/>
   
       <bean id="person" class="cn.waston.pojo.Person"/>
   
       <context:annotation-config/>
   </beans>
   ```

   ```java
   public class Person {
   
       @Autowired
       private Cat cat;
       @Autowired
       private Dog dog;
       private String name;
   
       public Cat getCat() {
           return cat;
       }
   
       public void setCat(Cat cat) {
           this.cat = cat;
       }
   
       public Dog getDog() {
           return dog;
       }
   
       public void setDog(Dog dog) {
           this.dog = dog;
       }
   
       public String getName() {
           return name;
       }
   
       public void setName(String name) {
           this.name = name;
       }
   
       @Override
       public String toString() {
           return "Person{" +
                   "cat=" + cat +
                   ", dog=" + dog +
                   ", name='" + name + '\'' +
                   '}';
       }
   }
   ```

   `@Autowired`可以在属性上使用，也可以在set方法上使用，还可以再加一个`@Qualifier(value = "xxx")`，来指定beanId为xxx的bean注入

   ps：使用了注解后，类中可以没有set方法，因为是通过**反射**的方式进行装配

   由J2EE提供的`@Resource`也可以用于自动装配，`@Resource` 可以通过`@Resource(name = "xxx")`的方式指定beanId为xxx的bean注入

3. 隐式的自动装配

   xml指定需要扫描的路径

   ```xml
   <?xml version="1.0" encoding="UTF-8"?>
   <beans xmlns="http://www.springframework.org/schema/beans"
          xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
          xmlns:context="http://www.springframework.org/schema/context"
          xmlns:aop="http://www.springframework.org/schema/aop"
          xsi:schemaLocation="http://www.springframework.org/schema/beans
           http://www.springframework.org/schema/beans/spring-beans.xsd
           http://www.springframework.org/schema/context
           http://www.springframework.org/schema/context/spring-context.xsd
           http://www.springframework.org/schema/aop
           http://www.springframework.org/schema/aop/spring-aop.xsd">
   
       <!--扫描指定路径下的注解-->
       <context:component-scan base-package="cn.waston.pojo"/>
       <context:annotation-config/>
   </beans>
   ```

   加上`@Component`的类被扫描到时，都会自动创建一个BeanId为类名小写的bean实例，并注入到IoC容器中

   ```JAVA
   @Component
   public class Cat {
   
   }
   
   @Component
   public class Dog {
   
   }
   
   @Component
   public class Person {
   
       @Resource
       private Cat cat;
       @Resource
       private Dog dog;
       @Value("sam")
       private String name;
   
       public Cat getCat() {
           return cat;
       }
   
       public Dog getDog() {
           return dog;
       }
   
       public String getName() {
           return name;
       }
   
       public void setName(String name) {
           this.name = name;
       }
   
       @Override
       public String toString() {
           return "Person{" +
                   "cat=" + cat +
                   ", dog=" + dog +
                   ", name='" + name + '\'' +
                   '}';
       }
   }
   
   public class TestAutowire {
   
       @Test
       public void testAutowire(){
           ApplicationContext context = new ClassPathXmlApplicationContext("context.xml");
           Person person = (Person) context.getBean("person");
           System.out.println(person.toString());
       }
   }
   ```

   `@Component`相当于

   ```
   <bean id="xxx" class="cn.waston.pojo.Xxx">
   ```

   `@Value`相当于

   ```
   <property name="xxx" value="xxx"></property>
   ```

   ,该注解也能用在set方法上

##### 用Java的方式配置Spring——JavaConfig

JavaConfig是Spring的一个子项目，使用它，可以做到Spring项目的全注解式开发（无xml文件）

例如：

```java
@Configuration
@ComponentScan("cn.waston.pojo")
public class MyConfig {
}
```

相当于

```xml
<beans ...>
    <context:component-scan base-package="cn.waston.pojo"/>
    <context:annotation-config/>
</beans>
```

测试：

```java
public class MyTest {
    public static void main(String[] args) {
        ApplicationContext context = new AnnotationConfigApplicationContext(MyConfig.class);
        Person person = (Person) context.getBean("person");
        System.out.println(person);
    }
}
```

`@Bean`注解的使用

```java
@Configuration
public class MyConfig {

    @Bean
    public Cat cat(){
        Cat cat = new Cat();
        cat.setName("lucy");
        return cat;
    }
}
```

相当于：

```xml
<beans ...>
    <bean id="cat" class="cn.waston.pojo.Cat">
        <property name="name" value="lucy"></property>
    </bean>
</beans>    
```

##### Bean的作用域

Spring支持五种作用域：

1. singleton：IoC容器中仅存在一个Bean实例（单例），默认作用域
2. prototype：每次从容器中调用Bean时，都返回一个新的实例，即每次调用getBean()时，都相当于执行new XxxBean()
3. request：每次HTTP请求都会创建一个新的Bean实例，仅适用于WebApplicationContext环境
4. session：同一个HTTP Session共享一个Bean实例，不同Session使用不同Bean， 仅适用于WebApplicationContext环境
5. globalSession：一般用于Portlet应用环境

设置作用域的方式是scope关键字/注解设置：

```xml
<bean id="userService" class="cn.waston.service.UserServiceImpl" p:userDao-ref="lowUserDao" scope="prototype"/>
```

```java
@Component
@Scope("prototype")
public class Cat {
}
```

#### 面向切面（AOP）

AOP为Aspect Oriented Programming的缩写，意为：面向切面编程，通过预编译方式和运行期**动态代理**实现程序功能的统一维护的一种技术。AOP是OOP的延续，是软件开发中的一个热点，也是Spring框架中的一个重要内容，是函数式编程的一种衍生范型。利用AOP可以对业务逻辑的各个部分进行隔离，从而使得业务逻辑各部分之间的耦合度降低，提高程序的可重用性，同时提高了开发的效率。

##### 代理模式

代理模式是Java设计模式的一种，也是SpringAOP的底层原理

代理模式的作用：将一些公共业务分离出来，由代理类执行以及集中管理（避免修改原有代码），降低耦合

代理模式的分类：

- 静态代理

  租房模型：

  ```java
  //租房接口
  public interface Rent {
      public void rent();
  }
  
  //房东 真实要出租房屋的角色
  public class Renter implements Rent{
      public void rent() {
          System.out.println("房东出租了房子");
      }
  }
  
  //中介 作为代理角色 帮助房东出租房屋
  public class Intermediary implements Rent{
      private Renter renter;
  
      public void rent() {
          //房东出租前需要看房
          System.out.println("带看房");
          renter.rent();
          //房东出租后需要签合同
          System.out.println("签合同");
      }
  
      public Intermediary(Renter renter) {
          this.renter = renter;
      }
  
      public Intermediary() {
      }
  }
  
  //租户 去租房子
  public class Tenant {
      public static void main(String[] args) {
          Renter renter = new Renter();
          //不直接找房东租，而是找中介租
          Intermediary intermediary = new Intermediary(renter);
          intermediary.rent();
      }
  }
  ```
  静态代理模式的缺点显而易见：每个真实类都需要一个代理类，代码量翻倍

- 动态代理

  与静态代理的不同是，其代理类是动态生成的，不是我们自己写好的

  动态代理的分类：基于接口（JDK动态代理）、基于类（cglib）、Java字节码实现（Javasist）

  JDK动态代理demo:

  ProxyInvocationHandler类（用于生成代理类）

  ```java
  import java.lang.reflect.InvocationHandler;
  import java.lang.reflect.Method;
  import java.lang.reflect.Proxy;
  
  public class ProxyInvocationHandler<T> implements InvocationHandler {
  
      //被代理的接口
      private T t;
  
      public void setT(T t) {
          this.t = t;
      }
  
      //生成得到代理类
      public Object getProxy(){
          return Proxy.newProxyInstance(this.getClass().getClassLoader(),
                  t.getClass().getInterfaces(), this);
      }
  
      //处理代理实例，并返回结果
      public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
  
          //利用反射机制，调用被代理的方法
          Object result = method.invoke(t, args);
          log();
  
          return result;
      }
  
      private void log(){
          System.out.println("生成日志");
      }
  }
  ```

  测试动态代理

  ```java
  public class Tenant {
      public static void main(String[] args) {
          Rent renter = new Renter();
  
          //通过调用程序处理角色来处理我们要调用的接口对象
          ProxyInvocationHandler<Rent> proxyInvocationHandler = new ProxyInvocationHandler<Rent>();
          proxyInvocationHandler.setT(renter);
          Rent proxy = (Rent) proxyInvocationHandler.getProxy();
          proxy.rent();
      }
  }
  ```

  相比于静态代理， 动态代理的优势在于可以很方便的对代理类的函数进行统一的处理，而不用修改每个代理类中的方法，如上面的log方法

  动态代理原理：可以看到代理类对象是通过调用Proxy类的newProxyInstance方法获取到的

  ```java
  /*loader: 用哪个类加载器去加载代理对象
  
  interfaces:动态代理类需要实现的接口
  
  h:动态代理方法在执行时，会调用h里面的invoke方法去执行*/
  public static Object newProxyInstance(ClassLoader loader,
                                        Class<?>[] interfaces,
                                        InvocationHandler h)
  ```

  当我们用代理对象执行任何方法时，都会去调用ProxyInvocationHandler类中的invoke方法，利用反射特性调用该方法，并执行invoke方法中的其他操作，如log()

##### Spring—AOP术语

- 横切关注点：跨越应用程序多个模块的，与我们的业务逻辑无关的，但我们需要关注的方法或功能，如日志、安全、缓存、事务等
- 切面(ASPECT)：横切关注点，被模块化的特殊对象，是一个类
- 通知(Advice)：切面必须要完成的工作，是类中的一个方法
- 目标(Target)：被通知对象
- 代理(Proxy)：向目标对象应用通知之后创建的对象
- 切入点(PointCut)：切面通知执行的“地点”的定义
- 连接点(jointPoint)：与切入点匹配的执行点

![](https://s1.imagehub.cc/images/2021/07/22/AOP748762641cf20235.jpg)

##### Spring—AOP的实现方式

首先需要配置SpringAOP的依赖

```xml
<dependency>
    <groupId>org.aspectj</groupId>
    <artifactId>aspectjweaver</artifactId>
    <version>1.9.4</version>
</dependency>
```

**实现方式一**：使用Spring的API接口+xml

业务接口

```java
public interface UserService {
    public void add();
    public void delete();
    public void update();
    public void select();
}
```

业务实现类

```java
public class UserServiceImpl implements UserService {
    public void add() {
        System.out.println("增加了一个用户");
    }

    public void delete() {
        System.out.println("删除了一个用户");
    }

    public void update() {
        System.out.println("更新了一个用户");
    }

    public void select() {
        System.out.println("查询了一个用户");
    }
}
```

切面类及通知

```java
public class BeforeLog implements MethodBeforeAdvice {
    //该方法会在目标方法调用前被调用
    public void before(Method method, Object[] objects, Object target) throws Throwable {
        System.out.println(target.getClass().getName() + "的" + method.getName() + "将被执行");
    }
}
```

```java
public class AfterLog implements AfterReturningAdvice {
    //该方法会在目标方法调用后被调用
    public void afterReturning(Object returnValue, Method method, Object[] objects, Object target) throws Throwable {
        System.out.println(target.getClass().getName() + "的" + method.getName() + "被执行了，返回值为" + returnValue);
    }
}
```

xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:aop="http://www.springframework.org/schema/aop"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/context
        http://www.springframework.org/schema/context/spring-context.xsd
        http://www.springframework.org/schema/aop
        http://www.springframework.org/schema/aop/spring-aop.xsd">

    <bean id="userService" class="cn.waston.service.UserServiceImpl"/>
    <bean id="beforeLog" class="cn.waston.log.BeforeLog"/>
    <bean id="afterLog" class="cn.waston.log.AfterLog"/>

    <!--配置AOP-->
    <aop:config>
        <!--切入点 expression表达式，用execution函数来表示要执行的位置-->
        <aop:pointcut id="pointcut" expression="execution(* cn.waston.service.UserServiceImpl.*(..))"/>

        <!--执行环绕增加-->
        <aop:advisor advice-ref="beforeLog" pointcut-ref="pointcut"/>
        <aop:advisor advice-ref="afterLog" pointcut-ref="pointcut"/>
    </aop:config>
</beans>
```

测试

```java
public class MyTest {
    public static void main(String[] args) {
        ClassPathXmlApplicationContext context = new ClassPathXmlApplicationContext("context.xml");
        UserService userService = (UserService) context.getBean("userService");
        userService.add();
    }
}
```

**方式二**：自定义类实现

自定切面类及其通知

```java
public class MyPointCut {
    public void before(){
        System.out.println("-----方法执行前-----");
    }

    public void after(){
        System.out.println("-----方法执行后-----");
    }
}
```

xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:aop="http://www.springframework.org/schema/aop"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/context
        http://www.springframework.org/schema/context/spring-context.xsd
        http://www.springframework.org/schema/aop
        http://www.springframework.org/schema/aop/spring-aop.xsd">

    <bean id="userService" class="cn.waston.service.UserServiceImpl"/>
    <bean id="beforeLog" class="cn.waston.log.BeforeLog"/>
    <bean id="afterLog" class="cn.waston.log.AfterLog"/>

    <bean id="myPointCut" class="cn.waston.my.MyPointCut"/>
    <aop:config>
        <!--自定义切面-->
        <aop:aspect ref="myPointCut">
            <!--切入点-->
            <aop:pointcut id="point" expression="execution(* cn.waston.service.UserServiceImpl.*(..))"/>
            <!--通知-->
            <aop:before method="before" pointcut-ref="point"/>
            <aop:after method="after" pointcut-ref="point"/>
        </aop:aspect>
    </aop:config>
</beans>
```

**方式三**：注解式

注解`@Aspect`可以使一个自定义类成为切面类

```java
import org.aspectj.lang.ProceedingJoinPoint;
import org.aspectj.lang.Signature;
import org.aspectj.lang.annotation.After;
import org.aspectj.lang.annotation.Around;
import org.aspectj.lang.annotation.Aspect;
import org.aspectj.lang.annotation.Before;

@Aspect
public class AnnotationPointCut {

    @Before("execution(* cn.waston.service.UserServiceImpl.*(..)))")
    public void before(){
        System.out.println("-----方法执行前-----");
    }

    @After("execution(* cn.waston.service.UserServiceImpl.*(..)))")
    public void after(){
        System.out.println("-----方法执行后-----");
    }

    //在环绕增强中，我们可以给定一个参数，代表我们要获取处理切入的点
    @Around("execution(* cn.waston.service.UserServiceImpl.*(..)))")
    public void around(ProceedingJoinPoint jp) throws Throwable {
        System.out.println("---环绕前---");

        //打印切入点签名
        Signature signature = jp.getSignature();
        System.out.println("signature:" + signature);

        //执行目标方法
        Object proceed = jp.proceed();
        System.out.println("---环绕后---");
    }
}
```

xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:aop="http://www.springframework.org/schema/aop"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/context
        http://www.springframework.org/schema/context/spring-context.xsd
        http://www.springframework.org/schema/aop
        http://www.springframework.org/schema/aop/spring-aop.xsd">

    <bean id="userService" class="cn.waston.service.UserServiceImpl"/>
    <bean id="beforeLog" class="cn.waston.log.BeforeLog"/>
    <bean id="afterLog" class="cn.waston.log.AfterLog"/>

    <bean id="annotationPointCut" class="cn.waston.my.AnnotationPointCut"/>
    <!--开启注解支持-->
    <aop:aspectj-autoproxy/>
</beans>
```



## SpringBoot

### SpringBoot简介

SpringBoot是在Spring的基础上发展起来的，它的出现旨在解决Spring工程**配置十分繁琐**的问题

它现在通常被选用为JavaWeb工程的**开发框架**，和SpringMVC类似，对比其他JavaWeb框架的优势是**约定大于配置**

它不是全新的框架，默认配置了很多框架的使用方式，就像是maven整合了所有的jar包一样，spring boot整合了所有的框架

**PS:微服务**

在当前流行的微服务架构中，SpringBoot是一个核心技术点

简单来说，微服务是一种架构风格，把一个应用构建成一系列小服务（模块）的组合；可以通过http（RPC）的方式进行互通

它打破了之前all in one的架构方式，把每个功能元素独立出来，把独立的功能元素进行动态组合，所以微服务架构是对**功能元素进行复制，而没有对整个应用进行复制**

​好处是：

1. 节省了调用资源
2. 每个功能元素的服务都是一个可替换的、可独立升级

### SpringBoot原理

讨论SpringBoot的原理，要先从其最具特点的自动配置原理开始

#### 自动配置

所有SpringBoot工程都基于一个Maven工程，在其pom.xml文件中有以下特点：

-  spring-boot-dependencies:核心依赖在父工程中！

-  我们在写或者引入一些SPringboot依赖的时候，不需要指定版本（web、test等），因为父依赖指定了版本

-  spring-boot-starter就是Springboot的启动场景（启动器）；比如spring-boot-starter-web，加上它就会自动导入web环境所有的依赖！

-  springboot会将所有的功能场景，都变成一个个的启动器

-  我们需要什么功能，只需要找到对应的启动器就可以了（Spring官网上有）​

**同时**，每个SpringBoot工程都有一个启动类

```java
@SpringBootApplication
public class XXXXApplication {
    public static void main(String[] args) {
        SpringApplication.run(XXXXApplication.class, args);
    }
}
```

工程通过执行run方法来启动，它做了四件事情：

1. 推断应用的类型是普通的项目还是Web项目
2. 查找并加载所有可用初始化器，设置到initializers属性中
3. 找出所有的应用程序监听器，设置到listeners属性中
4. 推断并设置main方法的定义类，找到运行的主类

那么，它是如何做到的呢？

可以看到，启动类上有如下的注解：（一个tab表示一层继承）

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

@Import通过快速导入的方式实现把实例加入spring的IoC容器中 ；**@Import注解可以用于导入第三方包** 

META-INF/spring.factories（org.springframework.boot:springboot:autoconfigure包中）：自动配置的核心文件；所有的自动配置都在这里面

**结论**：springboot所有自动配置都是在启动的时候扫描并加载（扫描上面的spring.factories），但不一定全部生效，要判断条件是否成立，只要导入了对应的start，就有对应的启动器了，有了启动器，自动配置就会生效，然后配置生效！

- springboot在启动的时候，从类路径下META-INF/spring.factories获取指定的值
- 将这些自动配置的类导入容器，自动配置就会生效，帮我们进行自动配置
- 以前需要我们自己配置的东西，现在springboot帮我们做了
- 整个javaEE解决方案和自动配置的东西都在spring-boot-autoconfigure-2.2.0.RELEASE.jar这个包下
- 它会把所有需要导入的组件，以类名的方式返回，这些组件就会被添加到容器
- 容器中也会存在非常多的xxxAutoConfiguration文件（都有@Configuration注解）（其中有很多@Bean），就是这些类给容器中导入了这个场景需要的所有组件，并自动配置
- 有了自动配置类，免去了我们手动编写配置文件的工作

##### 一段话解释SpringBoot自动配置原理

Spring Boot启动的时候会通过@EnableAutoConfiguration注解找到**META-INF/spring.factories**配置文件中的所有自动配置类，并对其进行加载，而这些自动配置类都是以AutoConfiguration结尾来命名的，它实际上就是一个JavaConfig形式的Spring容器配置类，它能通过以Properties结尾命名的类中取得在全局配置文件中配置的属性如：server.port，而XxxxProperties类是通过@ConfigurationProperties注解与全局配置文件中对应的属性进行绑定的。 

**自动配置的精髓所在**：

- SpringBoot启动会加载大量的自动配置类
- 我们看我们需要的功能有没有在SpringBoot默认写好的自动配置类当中
- 再来看这个自动配置类中到底配置了哪些组件（只要我们要用的组件存在其中，我们就不需要再去手动配置了）
- 自动配置类给容器中添加组件的时候，会从properties类中获取某些属性，我们只需要在配置文件中指定这些属性即可
- xxxAutoConfigurartion:自动配置类；给容器中添加组件
- xxxProperties：封装配置文件中的相关属性

#####  **自定义配置**

1、application.properties/application.yaml文件中去写，可以修改SpringBoot自动配置的默认值

- yaml文件还可以给实体类赋值：实体类上加@ConfigurationProperties(prefix = "yaml中的对象名")
- properties文件给实体类属性赋值：类上加@PropertySource(value = "classpath:xxx.properties")，属性值上加@Value("${properties里的属性名}")
- yaml具备松散绑定，yaml里写last-name，类属性是lastName，也能赋值

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

##### **多环境配置**

- 配置文件优先级：项目路径下config包内 > 项目路径下 > classpath/config > classpath
- **一般springboot项目里classpath目录指的是resources文件夹**
- 新建一个springboot项目，默认是最低的优先级
- 项目打包后，启动jar包时指定配置，会覆盖配置文件里的内容
- **多环境配置**：application.properties文件中写：**spring.profiles.active=application-xxx.properties**,启动时则以xxx配置文件中的内容为项目进行配置

##### 配置文件能写什么东西

- 找到**org.springframework.boot:springboot:autoconfigure**包，打开里面的META-INF/spring.factories文件，找到XXXConfiguration的源文件，再找到@EnableConfigurationProperties({yyy.class})中yyy的源文件，可以找到@ConfigurationProperties(    prefix = "zzz"）
- 这时就可以在application.properties文件中写zzz.ppp=kkk...来进行项目的相关配置了
- ppp就是zzz类中的属性
- XXXConfiguration类一般还有@ConditionalOnxxx如@ConditionalOnWebApplication(type = Type.SERVLET)等注解，表示生效场景，OnWebApplication表示只有是web场景（在pom.xml文件中写spring-boot-starter-web）时，后面的配置才会被载入，才会生效
- 每一个xxxAutoConfiguration类（配置类）都是容器中的一个组件，最后都加入到容器中；用它们来做自动配置
- 一旦某个配置类生效，它就会给容器中添加各种组件，这些组件的属性从对应的properties类中获取的，这些类里面的每一个属性又是和配置文件绑定的
- 所以我们可以在自己的配置文件中修改这些属性，从而改变系统的配置

**@Conditional：必须是@Conditional指定的条件成立，才给容器中添加组件，配置类里面的所有内容才生效！**

【**如何看自己项目中的哪些配置生效了**】：在配置文件中写debug=true，运行项目，在控制台（日志）可以看到

##### 后端校验

JSR-303校验

- 实体类上加@Validated；属性上加@Email()等JSR-303校验注解（constraints类中有）

- 有时需要引入

  ```java
  <dependency>
       <groupId>org.hibernate.validator</groupId>
       <artifactId>hibernate-validator</artifactId>
       <version>6.0.0.Final</version>
  </dependency>
  ```

  

