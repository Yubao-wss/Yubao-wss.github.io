---
layout: post
title: MyBatisPlus的使用
categories:
  - ACM
  - Template
tags: Posts
date: 2020-11-10 14:00:00
---

### MyBatisPlus是什么

- MyBatis的增强工具，为简化开发而生
- 所有的CRUD代码都可以由它自动生成，节省时间
- 与JPA、tk-mapper属于同一种类型的工具

### MyBatisPlus特性

- **无侵入**：只做增强不做改变，引入它不会对现有工程产生影响，如丝般顺滑
- **损耗小**：启动即会自动注入基本 CURD，性能基本无损耗，直接面向对象操作
- **强大的 CRUD 操作**：内置通用 Mapper、通用 Service，仅仅通过少量配置即可实现单表大部分 CRUD 操作，更有强大的条件构造器，满足各类使用需求
- **支持 Lambda 形式调用**：通过 Lambda 表达式，方便的编写各类查询条件，无需再担心字段写错
- **支持主键自动生成**：支持多达 4 种主键策略（内含分布式唯一 ID 生成器 - Sequence），可自由配置，完美解决主键问题
- **支持 ActiveRecord 模式**：支持 ActiveRecord 形式调用，实体类只需继承 Model 类即可进行强大的 CRUD 操作
- **支持自定义全局通用操作**：支持全局通用方法注入（ Write once, use anywhere ）
- **内置代码生成器**：采用代码或者 Maven 插件可快速生成 Mapper 、 Model 、 Service 、 Controller 层代码，支持模板引擎，更有超多自定义配置等您来使用
- **内置分页插件**：基于 MyBatis 物理分页，开发者无需关心具体操作，配置好插件之后，写分页等同于普通 List 查询
- **分页插件支持多种数据库**：支持 MySQL、MariaDB、Oracle、DB2、H2、HSQL、SQLite、Postgre、SQLServer 等多种数据库
- **内置性能分析插件**：可输出 Sql 语句以及其执行时间，建议开发测试时启用该功能，能快速揪出慢查询
- **内置全局拦截插件**：提供全表 delete 、 update 操作智能分析阻断，也可自定义拦截规则，预防误操作

### MyBatisPlus的使用

1. 创建用于测试的数据库、表及数据

2. 创建springboot项目，添加依赖如下：

   ```xml
   <dependency>
       <groupId>mysql</groupId>
       <artifactId>mysql-connector-java</artifactId>
   </dependency>
   
   <dependency>
       <groupId>org.projectlombok</groupId>
       <artifactId>lombok</artifactId>
   </dependency>
   
   <dependency>
       <groupId>com.baomidou</groupId>
       <artifactId>mybatis-plus-boot-starter</artifactId>
       <version>3.0.5</version>
   </dependency>
   ```

   尽量不要把mybatis与mybatis-plus依赖同时导入

3. 配置文件添加配置，连接数据库

   ```
   spring.datasource.username=root
   spring.datasource.password=password
   spring.datasource.url=jdbc:mysql://localhost:3306/mybatis?useSSL=false&useUnicode=true&characterEncoding=utf-8&serverTimezone=GMT%2B8
   spring.datasource.driver-class-name=com.mysql.cj.jdbc.Driver
   ```

4. 创建与数据库表对应的pojo类

   类中属性应该与数据库字段时刻保持一致（名称）

   补充：1、pojo类缺一些字段不会报错，但多了绝对不行

   2、属性驼峰命名可以对应表的“_”：如userName对user_name

   ```java
   package cn.waston.test_mp.pojo;
   
   import lombok.AllArgsConstructor;
   import lombok.Data;
   import lombok.NoArgsConstructor;
   
   import java.util.Date;
   
   /**
    * @Description:
    * @Author: Waston
    * @Date: 2020/11/11 9:33
    */
   @Data
   @AllArgsConstructor
   @NoArgsConstructor
   public class User {
       @TableId(type = IdType.AUTO)
       private Long id;
       private String username;
       private Date birthday;
       private Integer sex;
       private String address;
   }
   ```

5. 创建mapper接口，继承BaseMapper<T>

   ```java
   package cn.waston.test_mp.mapper;
   
   import cn.waston.test_mp.pojo.User;
   import com.baomidou.mybatisplus.core.mapper.BaseMapper;
   import org.springframework.stereotype.Repository;
   
   /**
    * @Description:
    * @Author: Waston
    * @Date: 2020/11/11 13:17
    */
   @Repository
   public interface UserMapper extends BaseMapper<User> {
   }
   ```

6. 启动类加注解，以扫描mapper

   ```java
   package cn.waston.test_mp;
   
   import org.mybatis.spring.annotation.MapperScan;
   import org.springframework.boot.SpringApplication;
   import org.springframework.boot.autoconfigure.SpringBootApplication;
   
   @MapperScan("cn.waston.test_mp.mapper")
   @SpringBootApplication
   public class TestMpApplication {
   
       public static void main(String[] args) {
           SpringApplication.run(TestMpApplication.class, args);
       }
   
   }
   ```

7. 测试调用BaseMapper<T>的方法

   ```java
   @SpringBootTest
   class TestMpApplicationTests {
       @Autowired
       private UserMapper userMapper;
   
       @Test
       void contextLoads() {
           List<User> users = userMapper.selectList(null);
           users.forEach(System.out::println);
       }
   
   }
   ```

#### 日志配置

- 配置文件种添加内容如下

  ```
  mybatis-plus.configuration.log-impl=org.apache.ibatis.logging.stdout.StdOutImpl
  ```

- 默认在控制台输出日志，会输出所执行的sql语句

#### CRUD

- 插入操作

  ```java
  @Test
  public void testInsert(){
      User user = new User();
      user.setUsername("sam");
      user.setAddress("beijing");
      user.setSex(1);
  
      int result = userMapper.insert(user);
      System.out.println(result);
  }
  ```

  result的值是1，代表受影响的行数

- 更新操作

  ```java
  @Test
      public void testUpdate(){
          User user = userMapper.selectById(28);
          user.setUsername("waston");
  
          userMapper.updateById(user);
      }
  ```

- 查询操作

  ```java
  @Test
  public void testQuery(){
      //单个查询
      User user = userMapper.selectById(29);
      //批量查询
      List<User> users = userMapper.selectBatchIds(Arrays.asList(28, 29, 30));
  
      //条件查询
      HashMap<String, Object> map = new HashMap<>();
      map.put("username","sam");
      List<User> users1 = userMapper.selectByMap(map);
  
  }
  ```

- 删除操作

  ```java
  @Test
  public void testDelete(){
      userMapper.deleteById(29);
      userMapper.deleteBatchIds(Arrays.asList(28,29,30));
  
      HashMap<String,Object> map = new HashMap<>();
      map.put("name","tom");
      userMapper.deleteByMap(map);
  }
  ```

#### 主键生成策略

MyBatisPlus默认使用雪花算法（相当于@TableId(type = IdType.ID_WORKER)），帮助我们自动生成id值，可以做到全局唯一

- 自增：@TableId(type = IdType.AUTO)

  必须保证数据库表的id是自增的，否则运行会报错

  同时，若表id设置为自增，采用AUTO之外的策略也会报错！

- 不使用：@TableId(type = IdType.NONE)

- 手动输入：@TableId(type = IdType.INPUT)

- UUID(全局唯一)：@TableId(type = IdType.UUID)

- @TableId(type = IdType.ID_WORDER_STR)：ID_WORKER的字符串表示法

#### 自动填充

一些数据例如：创建时间、修改时间；这些数据我们一般希望它们自动更新，有两种实现方式

- 方式一（通过数据库实现）

  1. 表中增加字段如：create_time、update_time
  2. 更新映射类
  3. 在进行增加、更新时，数据库就会自动把当前时间数据插入到表里所对应的字段中

- 方式二（MyBatisPlus实现）

  1. 映射类属性上增加注解

     ```java
     @TableField(fill = FieldFill.INSERT)
     private Date create_time;
     @TableField(fill = FieldFill.INSERT_UPDATE)
     private Date update_time;
     ```

  2. 编写处理器

     ```java
     package cn.waston.test_mp.handler;
     
     import com.baomidou.mybatisplus.core.handlers.MetaObjectHandler;
     import lombok.extern.slf4j.Slf4j;
     import org.apache.ibatis.reflection.MetaObject;
     import org.springframework.stereotype.Component;
     
     import java.util.Date;
     
     /**
      * @Description:
      * @Author: Waston
      * @Date: 2020/11/11 15:16
      */
     @Component
     @Slf4j
     public class MyMetaObjectHandler implements MetaObjectHandler {
         @Override
         public void insertFill(MetaObject metaObject) {
             this.setFieldValByName("createTime",new Date(),metaObject);
             this.setFieldValByName("updateTime",new Date(),metaObject);
         }
     
         @Override
         public void updateFill(MetaObject metaObject) {
             this.setFieldValByName("updateTime",new Date(),metaObject);
         }
     }
     ```

  3. 在进行增加、更新时，MyBatisPlus就会自动把当前时间数据插入到表里所对应的字段中

#### 乐观锁

乐观锁一般通过version（版本号）字段实现

- 乐观锁执行流程

  1. 取出记录时，获取到version
  2. 执行更新时，判断获取到的version是否等于当前表中的version
  3. 等于，则执行更新操作，并且version+1
  4. 不等于，不执行；重新取出记录，继续尝试更新（自旋锁实现）

- 实现步骤

  1. 表增加version字段，类型：int，default：1

  2. 映射类添加属性

     ```java
     @Version
     private Integer version;
     ```

  3. 注册组件

     ```java
     package cn.waston.test_mp.config;
     
     import com.baomidou.mybatisplus.extension.plugins.OptimisticLockerInterceptor;
     import org.mybatis.spring.annotation.MapperScan;
     import org.springframework.context.annotation.Bean;
     import org.springframework.context.annotation.Configuration;
     import org.springframework.transaction.annotation.EnableTransactionManagement;
     
     /**
      * @Description:
      * @Author: Waston
      * @Date: 2020/11/11 15:47
      */
     @MapperScan("cn.waston.test_mp.mapper")
     @EnableTransactionManagement
     @Configuration
     public class MyBatisPlusConfig {
         //注册乐观锁插件
         @Bean
         public OptimisticLockerInterceptor optimisticLockerInterceptor(){
             return new OptimisticLockerInterceptor();
         }
     }
     ```

  4. 这样在进行数据库操作的时候，MyBatisPlus就会去执行乐观锁的流程

     

#### 分页查询

MyBatisPlus内置了分页插件，底层通过limit实现分页查询

使用方法如下：

1. 注册分页插件

   ```java
   package cn.waston.test_mp.config;
   
   ...
   
   /**
    * @Description:
    * @Author: Waston
    * @Date: 2020/11/11 15:47
    */
   @MapperScan("cn.waston.test_mp.mapper")
   @EnableTransactionManagement
   @Configuration
   public class MyBatisPlusConfig {
       //分页插件
       public PaginationInterceptor paginationInterceptor(){
           PaginationInterceptor paginationInterceptor = new PaginationInterceptor();
           //设置请求的页面大于最大页后操作，true调回到首页，false继续请求 默认false
           //paginationInterceptor.setOverflow(false);
           //设置最大单页限制数量 默认500条 -1：不受限制
           //paginationInterceptor.setLimit(500);
           //开启 count 的join优化，只针对部分left join
           //paginationInterceptor.setCountSqlParser(new JsqlParserCountOptimize(true));
           return paginationInterceptor;
       }
   }
   
   ```

2. 测试分页查询

   ```java
   @Test
   public void testPage(){
       //参数1：页号  参数2：一页的数据条数
       Page<User> page = new Page<>(1,5);
       userMapper.selectPage(page,null);
   }
   ```

#### 逻辑删除

物理删除：从数据库中直接删除

逻辑删除：在数据库中没有直接删除，而是通过一个变量让其失效

**应用场景：**管理员可以查看被删除的内容、回收站

**使用方法：**

1. 表增加字段：deleted，类型：int，默认：0

2. 映射类增加属性

   ```java
   @TableLogic
   private Integer deleted;
   ```

3. 增加逻辑删除插件

   ```JAVA
   package cn.waston.test_mp.config;
   
   ...
   
   /**
    * @Description:
    * @Author: Waston
    * @Date: 2020/11/11 15:47
    */
   @MapperScan("cn.waston.test_mp.mapper")
   @EnableTransactionManagement
   @Configuration
   public class MyBatisPlusConfig {
       //逻辑删除插件
       @Bean
       public ISqlInjector sqlInjector(){
           return new LogicSqlInjector();
       }
   }
   ```

4. 配置文件中增加：

   ```
   mybatis-plus.global-config.db-config.logic-delete-value=1
   mybatis-plus.global-config.db-config.logic-not-delete-value=0
   ```

5. 这样，我们在执行一般删除操作时，会把deleted字段更新为1，而不是真正的删除

   而在执行一般查询操作时，MyBatisPlus则会把deleted加入到条件中进行查询

#### 性能分析插件

在遇到一些慢sql时，若超过了一定的时间，需要停止其执行

MyBatisPlus提供了一个性能分析插件，用于实现性能拦截器，输出每条sql语句及其执行时间

**使用方法：**

1. 注册该插件

   ```java
   package cn.waston.test_mp.config;
   
   ...
   /**
    * @Description:
    * @Author: Waston
    * @Date: 2020/11/11 15:47
    */
   @MapperScan("cn.waston.test_mp.mapper")
   @EnableTransactionManagement
   @Configuration
   public class MyBatisPlusConfig {
       //性能分析
       @Bean
       @Profile({"dev","test"})//限定环境
       public PerformanceInterceptor performanceInterceptor(){
           return new PerformanceInterceptor();
       }
   }
   ```

2. 这时，执行的sql时间超过限制就会终止，并且报错

#### 条件构造器Wrapper

在进行数据库操作时，所调用的函数如：selectList等，都需要传入一个参数，Wrapper类型，用于创建条件sql

**使用方法：**

```java
package cn.waston.test_mp;

import cn.waston.test_mp.mapper.UserMapper;
import cn.waston.test_mp.pojo.User;
import com.baomidou.mybatisplus.core.conditions.query.QueryWrapper;
import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;

import java.util.List;
import java.util.Map;

/**
 * @Description:
 * @Author: Waston
 * @Date: 2020/11/12 9:20
 */
@SpringBootTest
public class WrapperTest {
    @Autowired
    private UserMapper userMapper;

    @Test
    void testWrapper1(){
        //先构造QueryWrapper
        QueryWrapper<User> wrapper = new QueryWrapper<>();
        wrapper.isNotNull("username")//非空
                .ge("id",25l);//>=

        //构造条件查询
        userMapper.selectList(wrapper).forEach(System.out::println);
    }

    /**
     * 按名字单独查询一个
     * 若结果集有多个，selectOne会报错
     */
    @Test
    void testWrapper2(){
        QueryWrapper<User> wrapper = new QueryWrapper<>();
        wrapper.eq("username","tonny");

        //构造条件查询
        User user = userMapper.selectOne(wrapper);
    }

    /**
     * 区间查询
     */
    @Test
    void testWrapper3(){
        QueryWrapper<User> wrapper = new QueryWrapper<>();
        wrapper.between("id",20l,28l);

        Integer integer = userMapper.selectCount(wrapper);
        System.out.println(integer);
    }

    /**
     * 模糊查询
     */
    @Test
    void testWrapper4(){
        QueryWrapper<User> wrapper = new QueryWrapper<>();
        wrapper.notLike("address","x")
                .likeLeft("username","m");

        List<Map<String, Object>> maps = userMapper.selectMaps(wrapper);
        maps.forEach(System.out::println);
    }

    /**
     * 复杂查询
     */
    @Test
    void testWrapper5(){
        QueryWrapper<User> wrapper = new QueryWrapper<>();
        //构造一个in查询
        wrapper.inSql("id","select id from user where id<25");

        List<Object> objects = userMapper.selectObjs(wrapper);
        objects.forEach(System.out::println);
    }

    /**
     * 排序
     */
    @Test
    void testWrapper6(){
        QueryWrapper<User> wrapper = new QueryWrapper<>();
        //id降序
        wrapper.orderByDesc("id");

        List<Object> objects = userMapper.selectObjs(wrapper);
        objects.forEach(System.out::println);
    }
}
```

#### 代码自动生成

MyBatisPlus提供了一个代码生成器：AutoGenerator

通过它，我们可以快速生成Entity、Mapper、Mapper XML、Service、Controller等模块的代码

**使用方法：**

1. 添加依赖

   ```xml
   <dependency>
       <groupId>org.apache.velocity</groupId>
       <artifactId>velocity-engine-core</artifactId>
       <version>2.0</version>
   </dependency>
   ```

2. 构建代码生成器类

   ```java
   package cn.waston.test_mp.tools;
   
   import com.baomidou.mybatisplus.annotation.DbType;
   import com.baomidou.mybatisplus.annotation.FieldFill;
   import com.baomidou.mybatisplus.annotation.IdType;
   import com.baomidou.mybatisplus.generator.AutoGenerator;
   import com.baomidou.mybatisplus.generator.config.DataSourceConfig;
   import com.baomidou.mybatisplus.generator.config.GlobalConfig;
   import com.baomidou.mybatisplus.generator.config.PackageConfig;
   import com.baomidou.mybatisplus.generator.config.StrategyConfig;
   import com.baomidou.mybatisplus.generator.config.po.TableFill;
   import com.baomidou.mybatisplus.generator.config.rules.DateType;
   import com.baomidou.mybatisplus.generator.config.rules.NamingStrategy;
   
   import java.util.ArrayList;
   
   /**
    * @Description:
    * @Author: Waston
    * @Date: 2020/11/12 10:22
    */
   public class MyCode {
       public static void main(String[] args) {
           //new一个AutoGenerator对象
           AutoGenerator autoGenerator = new AutoGenerator();
   
           //代码生成路径
           GlobalConfig gc = new GlobalConfig();//全局配置类
           String projectPath = System.getProperty("user.dir");//获取当前项目路径
           gc.setOutputDir(projectPath + "/src/main/java");//配置具体路径
           gc.setAuthor("waston");//设置代码作者
           gc.setOpen(false);//是否打开路径文件夹
           gc.setFileOverride(false);//是否覆盖
           gc.setServiceName("%sService");//去除Service的I前缀
           gc.setIdType(IdType.AUTO);//id策略
           gc.setDateType(DateType.ONLY_DATE);//日期类型
           gc.setSwagger2(true);//配置swagger
           autoGenerator.setGlobalConfig(gc);
   
           //设置数据源
           DataSourceConfig dsc = new DataSourceConfig();
           dsc.setUrl("jdbc:mysql://localhost:3306/mybatis?useSSL=false&useUnicode=true&characterEncoding=utf-8&serverTimezone=GMT%2B8");
           dsc.setDriverName("com.mysql.cj.jdbc.Driver");
           dsc.setUsername("root");
           dsc.setPassword("password");
           dsc.setDbType(DbType.MYSQL);
           autoGenerator.setDataSource(dsc);
   
           //包名配置
           PackageConfig pc = new PackageConfig();
           pc.setModuleName("test_mp2");
           pc.setParent("cn.waston");
           pc.setEntity("entity");
           pc.setMapper("mapper");
           pc.setService("service");
           pc.setController("controller");
           autoGenerator.setPackageInfo(pc);
   
           //Entity配置
           StrategyConfig st = new StrategyConfig();
           st.setInclude("items");//要映射的表，可以传多个参数
           st.setNaming(NamingStrategy.underline_to_camel);//下划线转驼峰
           st.setColumnNaming(NamingStrategy.underline_to_camel);
           //st.setSuperEntityClass("");//自己的父类实体，没有就不用配置
           //st.setEntityLombokModel(true);//是否启用Lombok
           //st.setRestControllerStyle(true);//是否启用链式编程
           st.setLogicDeleteFieldName("deleted");//逻辑删除
           //自动填充策略
           TableFill createTime = new TableFill("create_time", FieldFill.INSERT);
           TableFill updateTime = new TableFill("update_time", FieldFill.INSERT_UPDATE);
           ArrayList<TableFill> tableFills = new ArrayList<>();
           tableFills.add(createTime);
           tableFills.add(updateTime);
           st.setTableFillList(tableFills);
           st.setVersionFieldName("version");//乐观锁
           st.setRestControllerStyle(true);//是否开启Controller RestFul风格
           st.setControllerMappingHyphenStyle(true);//请求url，下划线命名
           autoGenerator.setStrategy(st);
   
   
           autoGenerator.execute();//执行
       }
   }
   ```

3. 修改其中的配置，执行主方法