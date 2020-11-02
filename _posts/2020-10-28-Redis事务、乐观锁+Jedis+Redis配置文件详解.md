---
layout: post
title: Redis事务、乐观锁、持久化+Jedis+Redis配置文件详解
categories:
  - ACM
  - Template
tags: Posts
date: 2020-10-28 14:00:00
---

### 事务

#### 事务的概念

事务就是一组命令的集合；事务执行过程中，命令会按照顺序执行

事务需要具有  **一次性、顺序性、排他性**

#### Redis中的事务

- Redis的单条命令是保证原子性的，但其事务不保证原子性（命令要么全成功，要么全部失败）！

- Redis事务没有隔离级别的概念

- 事务中所有的命令入队后，不会直接执行，只有在发起执行命令时才会执行

- Redis事务的过程

  1. 开启事务

     redis-cli中执行命令：multi

  2. 命令入队

     输入get、set命令，回车，命令入队

  3. 执行事务

     执行命令：exec

- 放弃事务

  执行exec命令前，执行discard命令，即放弃事务，之前入队的命令全部取消xe

- 若任务队列中某条命令有语法错误，则事务中的所有命令都不会被执行

- 若队列执行时某条命令运行时出现异常，其他命令可以正常执行

### 乐观锁

#### 乐观锁的概念

- 悲观锁：多线程场景下，认为什么时候都可能出现问题，故无论做什么都会加锁
- 乐观锁：多线程场景下，任务大多数时候不会出现问题，所以不上锁！只在事务执行前后进行判断，如mysql中的version

#### Redis中的乐观锁

Redis中乐观锁通过监视测试功能实现

1. 执行事务前，redis-cli中执行：watch key_name，监视一个key
2. 若被监视的key中的值在事务执行前被改变（另一客户端），该事务执行则会失败，全部命令取消
3. 执行命令：unwatch，取消 watch 命令对所有 key 的监视 

### Jedis

#### Jedis是什么

Redis官方推荐的java连接开发工具

#### Jedis使用步骤

1. pom文件中加入相关依赖

   ```xml
   <dependency>
               <groupId>redis.clients</groupId>
               <artifactId>jedis</artifactId>
               <version>3.2.0</version>
           </dependency>
   
           <dependency>
               <groupId>com.alibaba</groupId>
               <artifactId>fastjson</artifactId>
               <version>1.2.62</version>
           </dependency>
   ```

2. 创建jedis对象，并测试连接

   ```java
   package cn.waston;
   
   import redis.clients.jedis.Jedis;
   
   public class Test {
       public static void main(String[] args) {
           //1、创建Jedis对象
           Jedis jedis = new Jedis("192.168.159.131",6379);
           //设置连接密码
           jedis.auth("123456");
           //2、测试连接
           System.out.println(jedis.ping());
       }
   }
   ```

3. 连接成功后，直接用jedis对象调用API，即可操作redis数据库

   常用API：<https://blog.csdn.net/zhangguanghui002/article/details/78770071> 

4. 最后jedis.close();断开连接

5. jedis执行事务

   ```java
   package cn.waston;
   
   import com.alibaba.fastjson.JSONObject;
   import redis.clients.jedis.Jedis;
   import redis.clients.jedis.Transaction;
   
   public class Test {
       public static void main(String[] args) {
           Jedis jedis = new Jedis("192.168.159.131",6379);
           jedis.auth("123456");
   
           JSONObject jsonObject = new JSONObject();
           jsonObject.put("id","01");
           jsonObject.put("name","waston");
           String result = jsonObject.toJSONString();
   
           //开启事务
           Transaction multi = jedis.multi();
           try {
               //命令入队
               multi.set("key1",result);
               multi.set("key2",result);
               //执行事务
               multi.exec();
           } catch (Exception e) {
               multi.discard();//异常，放弃事务
               e.printStackTrace();
           } finally {
               System.out.println(jedis.get("key1"));
               System.out.println(jedis.get("key2"));
               jedis.close();//关闭连接
           }
   
       }
   }
   ```

### Redis配置文件详解

#### Redis.conf

**通用配置GENERAL：**

- units are case insensitive so 1GB 1Gb 1gB are all the same：该文件对unit单位大小写不敏感

- include /.../xxx.conf：可以包含其他conf文件

- bind 127.0.0.1：redis绑定的ip

  若要让外网可以访问，一般设置为0.0.0.0

- protected-mode yes：是开启保护模式，默认为yes

- port 6379：端口号

- daemonize yes：是否以守护进程（后台方式2）启动，默认为no

  - 如果是yes，就需要pidfile /var/run/redis_6379.pid指定一个pid

- loglevel：日志级别，默认为notice（生产环境适用）

  - 级别排名：debug > verbose > notice > warning
  - logfile：日志路径

- databases：数据库数量，默认为16

- always-show-logo yes：是否显示logo

**快照SNAPSHOT：**

与RDB持久化相关

- save 900 1：如果900s内，至少1个key进行了修改，则Redis进行持久化

  save 300 10：300s内，至少10个key进行了修改，Redis进行持久化

  save 60 10000：60s内，至少10000个key进行了修改，Redis进行持久化

- stop-writes-on-bgsave-error yes：持久化出错后，Redis是否继续工作，默认是yes

- rdbcompression yes：是否压缩rdb文件，默认yes

- rdbchecksum yes：保存rdb文件时，是否进行错误检查校验，默认yes

- dir ./：rdb文件保存的目录，默认路径为Redis的启动目录（bin）

**REPLICATION:**

主从复制相关

- role：当前server的角色
- connection_slaves：从机数量

**SECURITY:**

安全相关

- requirepass xxx：密码，默认不开启

**CLIENTS:**

限制相关

- maxlients 10000：最大客户端连接数
- maxmemory <bytes>：最大内存容量
- maxmemory-policy noeviction：内存达到上限之后的处理策略
  1. volatile-lru：只对设置了过期时间的key进行lru删除
  2. allkeys-lru：删除lru算法的key
  3. volatile-random：随机删除即将过期的key
  4. allkeys-random：随机删除
  5. volatile-ttl：删除即将过期的
  6. noeviction：永不过期，报错

**APPEND ONLY MODE:**

AOF持久化相关

- appendonly no：默认不开启AOF模式

- appendfilename "appendonly.aof"：持久化文件的名字

- appendfsync：sync（同步）方式

  1. always：每次修改都进行sync
  2. everysec：每秒执行一次sync
  3. no：不执行sync

- auto-aof-rewrite-percentage 100

  auto-aof-rewrite-min-size 64mb

  当aof文件大于64m时，Redis会fork一个子线程重写aof文件，以优化其大小

### 持久化

持久化指的是Redis数据库将内存中的数据，保存到磁盘，否则Redis服务进程一旦关闭，数据都会消失！

Redis的持久化方式有两种：RDB和AOF

#### RDB(Redis DataBase)

Redis默认的持久化方式

**特点：**

- Redis会在指定时间间隔内将内存中的数据集快照写入磁盘
- Redis启动时，自动寻找启动目录（conf所在目录）下的dump.rdb文件，将其中的数据恢复到内存中
- Redis会单独创建（fork）一个子进程来进行持久化，先将数据写入一个临时文件中，待持久化过程结束后，再用这个临时文件替换上次持久化好的文件，**消耗资源**
- 整个过程中，主进程不进行任何IO操作，确保**高性能**
- 在进行大规模数据恢复时，数据完整性不能保证
- 最后一次持久化后的数据可能丢失（宕机）
- 默认保存的文件为**dump.rdb**

**触发机制：**

- save规则满足时
- 执行flushall命令后
- exit退出redis后

#### AOF(Append Only File)

该方式默认是关闭的，需要手动配置开启

**特点：**

- 该机制会把我们执行过的所有命令记录下来（读操作除外），Redis启动时，会把这个文件执行一遍
- 以日志的形式记录所有写操作，只追加文件，不会改写文件
- 默认保存的文件为**appendonly.aof**
- 可以用命令：redis-check-aof --fix appendonly.aof 修复aof文件，否则在aof文件存在语法错误的情况下，无法启动Redis
- AOF文件大小大于RDB，AOF恢复数据的效率也慢于RDB
- 该方式持续进行IO操作，**更消耗性能**

#### 扩展

- 同时开启两种持久化方式，Redis会优先载入AOF文件来恢复数据，因为AOF通常比RDB文件保存的更为完整
- 建议只在从机上持久化RDB文件，15分钟备份一次就好，只保留save 900 1这条规则