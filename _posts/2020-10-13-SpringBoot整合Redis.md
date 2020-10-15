---
layout: post
title: SpringBoot整合Redis
categories:
  - ACM
  - Template
tags: Posts
date: 2020-10-13 14:00:00
---

#### SpringBoot中整合Redis

1. 第一步pom中加入依赖

   ```xml
   <dependency>
       <groupId>org.springframework.boot</groupId>
       <artifactId>spring-boot-starter-data-redis</artifactId>
   </dependency>
   ```

   **SpringBoot2.x之后，原来底层使用的jedis被替换为了lettuce**

   - jedis使用的是直连，多线程情境下，存在数据安全问题，但可以通过jedis pool连接池解决
   - lettuce采用netty，实例可以在多个线程中共享，不存在线程不安全的问题

   有关Redis的自动配置类RedisAutoConfiguration的源码中有

   ```java
   public class RedisAutoConfiguration {
       public RedisAutoConfiguration() {
       }
   
       @Bean
       @ConditionalOnMissingBean(
           name = {"redisTemplate"}//说明我们可以自己定义一个RedisTemplate
       )
       //默认的RedisTemplate两个泛型都是Object，在使用时需要强转为String,Object，不方便，而我们可以通过自定义RedisTemplate以替换之
       public RedisTemplate<Object, Object> redisTemplate(RedisConnectionFactory redisConnectionFactory) throws UnknownHostException {
           RedisTemplate<Object, Object> template = new RedisTemplate();
           template.setConnectionFactory(redisConnectionFactory);
           return template;
       }
   
       @Bean
       @ConditionalOnMissingBean
       //spring为我们生成了String版的StringRedisTemplate，我们一般直接使用这个
       public StringRedisTemplate stringRedisTemplate(RedisConnectionFactory redisConnectionFactory) throws UnknownHostException {
           StringRedisTemplate template = new StringRedisTemplate();
           template.setConnectionFactory(redisConnectionFactory);
           return template;
       }
   }
   ```

   

2. 配置连接

   ```
   spring.redis.host=127.0.0.1
   spring.redis.port=6379
   ```

   

3. 测试

   ```java
   @SpringBootTest
   class RedisSpringbootApplicationTests {
   
       @Autowired
       private RedisTemplate redisTemplate;
   
       @Test
       void contextLoads() {
           //1、获取连接(不常用)
           //RedisConnection connection = redisTemplate.getConnectionFactory().getConnection();
   
           //2、数据操作
           //opsForxx, 给redisTemplate操作数据的类型，如opsForValue、opsForList等等
           //eg：redisTemplate.opsForList()
           //常用的其他方法都可以直接通过redisTemplate来进行，如CRUD、事务等
   
           redisTemplate.opsForValue().set("mykey","waston");
           System.out.println(redisTemplate.opsForValue().get("mykey"));
       }
   
   }
   ```
   - 默认redisTemplate中的序列化方式是JDK序列化，值为中文时，可能会被转义，所以我们需要自定义redisTemplate

     ```java
     @Configuration
     public class RedisConfig {
         @Bean
         public RedisTemplate<String, Object> redisTemplate(RedisConnectionFactory redisConnectionFactory) throws UnknownHostException {
             RedisTemplate<String, Object> template = new RedisTemplate();
             template.setConnectionFactory(redisConnectionFactory);
     
             //配置序列化方式
     
             //jackson序列化
             Jackson2JsonRedisSerializer objectJackson2JsonRedisSerializer = new Jackson2JsonRedisSerializer(Object.class);
             //使用ObjectMapper转义
             ObjectMapper om = new ObjectMapper();
             om.setVisibility(PropertyAccessor.ALL, JsonAutoDetect.Visibility.ANY);
             om.enableDefaultTyping(ObjectMapper.DefaultTyping.NON_FINAL);
             objectJackson2JsonRedisSerializer.setObjectMapper(om);
     
             //String的序列化
             StringRedisSerializer stringRedisSerializer = new StringRedisSerializer();
     
             //key采用String的序列化方式
             template.setKeySerializer(stringRedisSerializer);
             //hash的key也采用String的序列化方式
             template.setHashKeySerializer(stringRedisSerializer);
             //value序列化方式采用jackson
             template.setValueSerializer(objectJackson2JsonRedisSerializer);
             //hash的value序列化方式采用jackson
             template.setHashValueSerializer(objectJackson2JsonRedisSerializer);
     
             //set所有的Properties
             template.afterPropertiesSet();
             return template;
         }
     }
     ```

   - 也可以先通过json序列化，再进行数据操作，以避免上述问题

     ```java
     @Test
     void test() throws JsonProcessingException {
         //new一个实体类对象
         User user = new User("哈哈哈", 18);
         //序列化该对象
         String jsonUser = new ObjectMapper().writeValueAsString(user);
         //数据操作
         redisTemplate.opsForValue().set("user",jsonUser);
         Object user1 = redisTemplate.opsForValue().get("user");
         System.out.println(user1);
     }
     ```

   

4. Redis工具类：RedisUtils

   ```java
   package cn.waston.redisspringboot.utils;
   
   import org.springframework.beans.factory.annotation.Autowired;
   import org.springframework.data.redis.core.RedisTemplate;
   import org.springframework.stereotype.Component;
   import org.springframework.util.CollectionUtils;
   
   import java.util.List;
   import java.util.Map;
   import java.util.Set;
   import java.util.concurrent.TimeUnit;
   
   /**
    * @Description:
    * @Author: Waston
    * @Date: 2020/10/14 11:32
    */
   @Component
   public class RedisUtil {
   
       @Autowired
       private RedisTemplate<String, Object> redisTemplate;
       // =============================common============================
       /**
        * 指定缓存失效时间
        * @param key 键
        * @param time 时间(秒)
        * @return
        */
       public boolean expire(String key, long time) {
           try {
               if (time > 0) {
                   redisTemplate.expire(key, time, TimeUnit.SECONDS);
               }
               return true;
           } catch (Exception e) {
               e.printStackTrace();
               return false;
           }
       }
       /**
        * 根据key 获取过期时间
        * @param key 键 不能为null
        * @return 时间(秒) 返回0代表为永久有效
        */
       public long getExpire(String key) {
           return redisTemplate.getExpire(key, TimeUnit.SECONDS);
       }
   
       /**
        * 判断key是否存在
        * @param key 键
        * @return true 存在 false不存在
        */
       public boolean hasKey(String key) {
           try {
               return redisTemplate.hasKey(key);
           } catch (Exception e) {
               e.printStackTrace();
               return false;
           }
       }
   
       /**
        * 删除缓存
        * @param key 可以传一个值 或多个
        */
       @SuppressWarnings("unchecked")
       public void del(String... key) {
           if (key != null && key.length > 0) {
               if (key.length == 1) {
                   redisTemplate.delete(key[0]);
               } else {
                   redisTemplate.delete(CollectionUtils.arrayToList(key));
               }
           }
       }
       // ============================String=============================
       /**
        * 普通缓存获取
        * @param key 键
        * @return 值
        */
       public Object get(String key) {
           return key == null ? null : redisTemplate.opsForValue().get(key);
       }
   
       /**
        * 普通缓存放入
        * @param key 键
        * @param value 值
        * @return true成功 false失败
        */
       public boolean set(String key, Object value) {
           try {
               redisTemplate.opsForValue().set(key, value);
               return true;
           } catch (Exception e) {
               e.printStackTrace();
               return false;
           }
       }
   
       /**
        * 普通缓存放入并设置时间
        * @param key 键
        * @param value 值
        * @param time 时间(秒) time要大于0 如果time小于等于0 将设置无限期
        * @return true成功 false 失败
        */
       public boolean set(String key, Object value, long time) {
           try {
               if (time > 0) {
                   redisTemplate.opsForValue().set(key, value, time, TimeUnit.SECONDS);
               } else {
                   set(key, value);
               }
               return true;
           } catch (Exception e) {
               e.printStackTrace();
               return false;
           }
       }
       /**
        * 递增
        * @param key 键
        * @param delta 要增加几(大于0)
        * @return
        */
       public long incr(String key, long delta) {
           if (delta < 0) {
               throw new RuntimeException("递增因子必须大于0");
           }
           return redisTemplate.opsForValue().increment(key, delta);
       }
   
       /**
        * 递减
        * @param key 键
        * @param delta 要减少几(小于0)
        * @return
        */
       public long decr(String key, long delta) {
           if (delta < 0) {
               throw new RuntimeException("递减因子必须大于0");
           }
           return redisTemplate.opsForValue().increment(key, -delta);
       }
   
       // ================================Map=================================
       /**
        * HashGet
        * @param key 键 不能为null
        * @param item 项 不能为null
        * @return 值
        */
       public Object hget(String key, String item) {
           return redisTemplate.opsForHash().get(key, item);
       }
   
       /**
        * 获取hashKey对应的所有键值
        * @param key 键
        * @return 对应的多个键值
        */
       public Map<Object, Object> hmget(String key) {
           return redisTemplate.opsForHash().entries(key);
       }
   
       /**
        * HashSet
        * @param key 键
        * @param map 对应多个键值
        * @return true 成功 false 失败
        */
       public boolean hmset(String key, Map<String, Object> map) {
           try {
               redisTemplate.opsForHash().putAll(key, map);
               return true;
           } catch (Exception e) {
               e.printStackTrace();
               return false;
           }
       }
       /**
        * HashSet 并设置时间
        * @param key 键
        * @param map 对应多个键值
        * @param time 时间(秒)
        * @return true成功 false失败
        */
       public boolean hmset(String key, Map<String, Object> map, long time) {
           try {
               redisTemplate.opsForHash().putAll(key, map);
               if (time > 0) {
                   expire(key, time);
               }
               return true;
           } catch (Exception e) {
               e.printStackTrace();
               return false;
           }
       }
   
       /**
        * 向一张hash表中放入数据,如果不存在将创建
        * @param key 键
        * @param item 项
        * @param value 值
        * @return true 成功 false失败
        */
       public boolean hset(String key, String item, Object value) {
           try {
               redisTemplate.opsForHash().put(key, item, value);
               return true;
           } catch (Exception e) {
               e.printStackTrace();
               return false;
           }
       }
       /**
        * 向一张hash表中放入数据,如果不存在将创建
        * @param key 键
        * @param item 项
        * @param value 值
        * @param time 时间(秒) 注意:如果已存在的hash表有时间,这里将会替换原有的时间
        * @return true 成功 false失败
        */
       public boolean hset(String key, String item, Object value, long time) {
           try {
               redisTemplate.opsForHash().put(key, item, value);
               if (time > 0) {
                   expire(key, time);
               }
               return true;
           } catch (Exception e) {
               e.printStackTrace();
               return false;
           }
       }
   
       /**
        * 删除hash表中的值
        * @param key 键 不能为null
        * @param item 项 可以使多个 不能为null
        */
   
       public void hdel(String key, Object... item) {
           redisTemplate.opsForHash().delete(key, item);
       }
   
       /**
        * 判断hash表中是否有该项的值
        * @param key 键 不能为null
        * @param item 项 不能为null
        * @return true 存在 false不存在
        */
       public boolean hHasKey(String key, String item) {
           return redisTemplate.opsForHash().hasKey(key, item);
       }
   
       /**
        * hash递增 如果不存在,就会创建一个 并把新增后的值返回
        * @param key 键
        * @param item 项
        * @param by 要增加几(大于0)
        * @return
        */
       public double hincr(String key, String item, double by) {
           return redisTemplate.opsForHash().increment(key, item, by);
       }
       /**
        * hash递减
        * @param key 键
        * @param item 项
        * @param by 要减少记(小于0)
        * @return
        */
       public double hdecr(String key, String item, double by) {
           return redisTemplate.opsForHash().increment(key, item, -by);
       }
       // ============================set=============================
       /**
        * 根据key获取Set中的所有值
        * @param key 键
        * @return
        */
       public Set<Object> sGet(String key) {
           try {
               return redisTemplate.opsForSet().members(key);
           } catch (Exception e) {
               e.printStackTrace();
               return null;
           }
       }
   
       /**
        * 根据value从一个set中查询,是否存在
        * @param key 键
        * @param value 值
        * @return true 存在 false不存在
        */
       public boolean sHasKey(String key, Object value) {
           try {
               return redisTemplate.opsForSet().isMember(key, value);
           } catch (Exception e) {
               e.printStackTrace();
               return false;
           }
       }
       /**
        * 将数据放入set缓存
        * @param key 键
        * @param values 值 可以是多个
        * @return 成功个数
        */
       public long sSet(String key, Object... values) {
           try {
               return redisTemplate.opsForSet().add(key, values);
           } catch (Exception e) {
               e.printStackTrace();
               return 0;
           }
       }
   
       /**
        * 将set数据放入缓存
        * @param key 键
        * @param time 时间(秒)
        * @param values 值 可以是多个
        * @return 成功个数
        */
       public long sSetAndTime(String key, long time, Object... values) {
           try {
               Long count = redisTemplate.opsForSet().add(key, values);
               if (time > 0)
                   expire(key, time);
               return count;
           } catch (Exception e) {
               e.printStackTrace();
               return 0;
           }
       }
   
       /**
        * 获取set缓存的长度
        * @param key 键
        * @return
        */
       public long sGetSetSize(String key) {
           try {
               return redisTemplate.opsForSet().size(key);
           } catch (Exception e) {
               e.printStackTrace();
               return 0;
           }
       }
   
       /**
        * 移除值为value的
        * @param key 键
        * @param values 值 可以是多个
        * @return 移除的个数
        */
       public long setRemove(String key, Object... values) {
           try {
               Long count = redisTemplate.opsForSet().remove(key, values);
               return count;
           } catch (Exception e) {
               e.printStackTrace();
               return 0;
           }
       }
       // ===============================list=================================
       /**
        * 获取list缓存的内容
        * @param key 键
        * @param start 开始
        * @param end 结束 0 到 -1代表所有值
        * @return
        */
       public List<Object> lGet(String key, long start, long end) {
           try {
               return redisTemplate.opsForList().range(key, start, end);
           } catch (Exception e) {
               e.printStackTrace();
               return null;
           }
       }
   
       /**
        * 获取list缓存的长度
        * @param key 键
        * @return
        */
       public long lGetListSize(String key) {
           try {
               return redisTemplate.opsForList().size(key);
           } catch (Exception e) {
               e.printStackTrace();
               return 0;
           }
       }
       /**
        * 通过索引 获取list中的值
        * @param key 键
        * @param index 索引 index>=0时， 0 表头，1 第二个元素，依次类推；index<0时，-1，表尾，-2倒数第二个元素，依次类推
        * @return
        */
       public Object lGetIndex(String key, long index) {
           try {
               return redisTemplate.opsForList().index(key, index);
           } catch (Exception e) {
               e.printStackTrace();
               return null;
           }
       }
   
       /**
        * 将list放入缓存
        * @param key 键
        * @param value 值
        * @return
        */
       public boolean lSet(String key, Object value) {
           try {
               redisTemplate.opsForList().rightPush(key, value);
               return true;
           } catch (Exception e) {
               e.printStackTrace();
               return false;
           }
       }
   
       /**
        * 将list放入缓存
        * @param key 键
        * @param value 值
        * @param time 时间(秒)
        * @return
        */
       public boolean lSet(String key, Object value, long time) {
           try {
               redisTemplate.opsForList().rightPush(key, value);
               if (time > 0)
                   expire(key, time);
               return true;
           } catch (Exception e) {
               e.printStackTrace();
               return false;
           }
       }
   
       /**
        * 将list放入缓存
        * @param key 键
        * @param value 值
        * @return
        */
       public boolean lSet(String key, List<Object> value) {
           try {
               redisTemplate.opsForList().rightPushAll(key, value);
               return true;
           } catch (Exception e) {
               e.printStackTrace();
               return false;
           }
       }
   
       /**
        * 将list放入缓存
        *
        * @param key 键
        * @param value 值
        * @param time 时间(秒)
        * @return
        */
       public boolean lSet(String key, List<Object> value, long time) {
           try {
               redisTemplate.opsForList().rightPushAll(key, value);
               if (time > 0)
                   expire(key, time);
               return true;
           } catch (Exception e) {
               e.printStackTrace();
               return false;
           }
       }
   
       /**
        * 根据索引修改list中的某条数据
        * @param key 键
        * @param index 索引
        * @param value 值
        * @return
        */
       public boolean lUpdateIndex(String key, long index, Object value) {
           try {
   
               redisTemplate.opsForList().set(key, index, value);
               return true;
           } catch (Exception e) {
               e.printStackTrace();
               return false;
           }
       }
       /**
        * 移除N个值为value
        * @param key 键
        * @param count 移除多少个
        * @param value 值
        * @return 移除的个数
        */
       public long lRemove(String key, long count, Object value) {
           try {
               Long remove = redisTemplate.opsForList().remove(key, count, value);
               return remove;
           } catch (Exception e) {
               e.printStackTrace();
               return 0;
           }
       }
   
   
       
   }
   
   ```

   