---
layout: post
title: spring boot中redis集群使用以及对象缓存
tags:
- redis
- spring boot
categories: redis
---
　　Redis是使用最广泛的内存数据存储，Redis支持更丰富的数据结构，例如hashes, lists, sets等，同时支持数据持久化，同时Redis还提供一些类数据库的特性，比如事务，HA，主从库。本文主要介绍redis的安装，以及结合spring boot来连接redis集群环境进行数据的处理，同时介绍redis通过Cacheable（缓存）、CachePut（更新缓存）、CacheEvict（删除缓存），希望对要在spring boot中使用redis或者用来缓存对象有一定的帮助。

<!-- more -->


####安装指南


下载地址：http://download.redis.io/releases/redis-3.2.6.tar.gz

解压到目录：

```
[root@dev opt]# tar -xzf redis-3.2.6.tar.gz
```

启动redis
```
[root@dev opt]# cd redis-3.2.6/src
[root@dev src]# ./redis-server &
```

如果出现如下的内容表示启动成功
```
                _._
           _.-``__ ''-._
      _.-``    `.  `_.  ''-._           Redis 3.2.6 (00000000/0) 64 bit
  .-`` .-```.  ```\/    _.,_ ''-._
 (    '      ,       .-`  | `,    )     Running in standalone mode
 |`-._`-...-` __...-.``-._|'` _.-'|     Port: 6379
 |    `-._   `._    /     _.-'    |     PID: 26258
  `-._    `-._  `-./  _.-'    _.-'
 |`-._`-._    `-.__.-'    _.-'_.-'|
 |    `-._`-._        _.-'_.-'    |           http://redis.io
  `-._    `-._`-.__.-'_.-'    _.-'
 |`-._`-._    `-.__.-'    _.-'_.-'|
 |    `-._`-._        _.-'_.-'    |
  `-._    `-._`-.__.-'_.-'    _.-'
      `-._    `-.__.-'    _.-'
          `-._        _.-'
              `-.__.-'

26258:M 10 Dec 15:43:37.249 # WARNING: The TCP backlog setting of 511 cannot be enforced because /proc/sys/net/core/somaxconn is set to the lower value of 128.
26258:M 10 Dec 15:43:37.250 # Server started, Redis version 3.2.6
26258:M 10 Dec 15:43:37.250 # WARNING you have Transparent Huge Pages (THP) support enabled in your kernel. This will create latency and memory usage issues with Redis. To fix this issue run the command 'echo never > /sys/kernel/mm/transparent_hugepage/enabled' as root, and add it to your /etc/rc.local in order to retain the setting after a reboot. Redis must be restarted after THP is disabled.
26258:M 10 Dec 15:43:37.250 * DB loaded from disk: 0.000 seconds
26258:M 10 Dec 15:43:37.250 * The server is now ready to accept connections on port 6379
```

连接redis并且保存数据
```
[root@dev src]# cd redis-3.2.6/src
[root@dev src]# ./redis-cli
127.0.0.1:6379> set test jianxw
OK
127.0.0.1:6379> get test
"jianxw"
```


####springboot连接redis集群和对数据进行操作

首先引入需要的jar包：
```java
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.jianxw</groupId>
    <artifactId>spring-boot-redis</artifactId>
    <version>1.0-SNAPSHOT</version>
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>1.4.2.RELEASE</version>
        <relativePath/>
        <!-- lookup parent from repository -->
    </parent>
    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-redis</artifactId>
        </dependency>
    </dependencies>

</project>
```

增加配置文件application.properties，同时增加redis的集群地址配置和其他配置：

```

# Redis数据库索引
spring.redis.database=0
# Redis集群服务器地址
spring.redis.cluster.nodes[0]=192.168.1.100:7000
spring.redis.cluster.nodes[1]=192.168.1.100:7001
spring.redis.cluster.nodes[2]=192.168.1.100:7002
spring.redis.cluster.nodes[3]=192.168.1.100:7003
spring.redis.cluster.nodes[4]=192.168.1.100:7004
# Redis服务器连接密码（默认为空）
spring.redis.password=
# 连接池最大连接数（使用负值表示没有限制）
spring.redis.pool.max-active=16
# 连接池最大阻塞等待时间（使用负值表示没有限制）
spring.redis.pool.max-wait=-1
# 连接池中的最大空闲连接
spring.redis.pool.max-idle=16
# 连接池中的最小空闲连接
spring.redis.pool.min-idle=0
# 连接超时时间（毫秒）
spring.redis.timeout=2000

```

使用spring-data-redis的RedisTemplate连接和使用redis：

```java

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.CommandLineRunner;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cache.annotation.EnableCaching;
import org.springframework.data.redis.core.StringRedisTemplate;
import org.springframework.data.redis.core.ValueOperations;
import org.springframework.stereotype.Component;

@SpringBootApplication
@EnableCaching
@Component(value = "com.jianxw.redis")
public class SampleRedisApplication implements CommandLineRunner {

    @Autowired
    private StringRedisTemplate template;

    @Override
    public void run(String... args) throws Exception {
        ValueOperations<String, String> ops = this.template.opsForValue();
        String key = "spring.boot.redis.test";
        if (!this.template.hasKey(key)) {
            ops.set(key, "jianxw");
        }
        System.out.println("redis里面的" + key + "对应的value为：" + ops.get(key));
    }

    public static void main(String[] args) throws Exception {
        SpringApplication.run(SampleRedisApplication.class, args);
    }

}

```

如果输出：“redis里面的spring.boot.redis.test对应的value为：jianxw”，表示redis的操作成功。


####通过Cacheable、CachePut、CacheEvict来进行缓存的操作

首先创建缓存的配置类，来设置缓存的时间等信息：

```java

import org.springframework.cache.CacheManager;
import org.springframework.cache.annotation.CachingConfigurerSupport;
import org.springframework.cache.annotation.EnableCaching;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.data.redis.cache.RedisCacheManager;
import org.springframework.data.redis.connection.RedisConnectionFactory;
import org.springframework.data.redis.core.RedisTemplate;

/**
 * redis的配置类
 * @author jianxw
 */

@Configuration
@EnableCaching
public class RedisConfig extends CachingConfigurerSupport {

    @SuppressWarnings("rawtypes")
    @Bean
    public CacheManager cacheManager(RedisTemplate redisTemplate) {
        RedisCacheManager redisCacheManager = new RedisCacheManager(redisTemplate);
        //设置缓存过期时间60s
        redisCacheManager.setDefaultExpiration(60);
        return redisCacheManager;
    }

    @Bean
    public RedisTemplate<Object, Object> redisTemplate(RedisConnectionFactory factory) {
        RedisTemplate<Object, Object> redisTemplate = new RedisTemplate<Object, Object>();
        redisTemplate.setConnectionFactory(factory);
        redisTemplate.setExposeConnection(true);
        return redisTemplate;
    }

}

```

创建user类：
```java

import org.springframework.data.annotation.Id;
import java.io.Serializable;

public class User implements Serializable {
    private static final long serialVersionUID = 1L;

    @Id
    private String id;

    private String name;
    private int age;


    public User() {
    }

    public User(String id, String name, int age) {
        this.name = name;
        this.age = age;
        this.id = id;
    }

    public User(int age) {
        this.age = age;
    }

    public String getId() {
        return id;
    }

    public void setId(String id) {
        this.id = id;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public int getAge() {
        return age;
    }

    public void setAge(int age) {
        this.age = age;
    }


    @Override
    public String toString() {
        return "User{" +
                "id='" + id + '\'' +
                ", name='" + name + '\'' +
                ", age=" + age +
                '}';
    }
}

```

创建UserService，实现IUserService来进行用户数据的增加、更新和删除的同时更新缓存里面的值：
```java

import org.springframework.cache.annotation.CacheEvict;
import org.springframework.cache.annotation.CachePut;
import org.springframework.cache.annotation.Cacheable;
import org.springframework.stereotype.Service;

/**
 * @author jianxw
 */
@Service
public class UserService implements IUserService {

    @Override
    @Cacheable(value = "users", key = "#id")
    public User findUserById(String id) {
        System.out.println("进入方法，不是从缓存中取到id为" + id +"的值");
        return getById(id);
    }

    @Override
    @CachePut(value = "users", key = "#id")
    public User updateUserById(String id) {
        return new User(id, "jianxw_update_" + id, 12);
    }

    @Override
    @CacheEvict(value = "users", key = "#id")
    public void deleteById(String id) {
        System.out.println("删除用户id为" + id + "的成功");
    }

    private User getById(String id) {
        return new User(id, "jianxw_" + id, 12);
    }

}

```

调用Service里面的方法进行用户数据信息的增加、更新和删除：

```java

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.CommandLineRunner;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cache.annotation.EnableCaching;
import org.springframework.data.redis.core.StringRedisTemplate;
import org.springframework.stereotype.Component;

import javax.annotation.Resource;

@SpringBootApplication
@EnableCaching
@Component(value = "com.jianxw.redis")
public class SampleRedisApplication implements CommandLineRunner {

    @Autowired
    private StringRedisTemplate template;

    @Resource
    private IUserService iUserService;

    @Override
    public void run(String... args) throws Exception {
        testUserCache();
        testUpdateUserCache();
        testDeleteUser();
    }

    public static void main(String[] args) throws Exception {
        SpringApplication.run(SampleRedisApplication.class, args);
    }

    private void testUserCache() {
        System.out.println("--------------开始通过id获取用户的信息-------------");
        System.out.println("获取用户id为4的信息：" + iUserService.findUserById("4"));
        System.out.println("获取用户id为5的信息：" + iUserService.findUserById("5"));
        System.out.println("获取用户id为4的信息：" + iUserService.findUserById("4"));
        System.out.println("--------------结束通过id获取用户的信息-------------");
    }


    private void testUpdateUserCache() {
        System.out.println("--------------开始更新用户id为4的信息-------------");
        System.out.println("更新后id为4的信息为：" + iUserService.updateUserById("4"));
        System.out.println("获取用户id为4的信息：" + iUserService.findUserById("4"));
        System.out.println("--------------结束更新用户id为4的信息-------------");
    }

    private void testDeleteUser() {
        System.out.println("--------------开始删除用户id为4的信息-------------");
        iUserService.deleteById("4");
        System.out.println("获取用户id为4的信息：" + iUserService.findUserById("4"));
        System.out.println("--------------结束删除用户id为4的信息-------------");
    }

}

```
运行结果为：

```

--------------开始通过id获取用户的信息-------------
进入方法，不是从缓存中取到id为4的值
获取用户id为4的信息：User{id='4', name='jianxw_4', age=12}
进入方法，不是从缓存中取到id为5的值
获取用户id为5的信息：User{id='5', name='jianxw_5', age=12}
获取用户id为4的信息：User{id='4', name='jianxw_4', age=12}
--------------结束通过id获取用户的信息-------------
--------------开始更新用户id为4的信息-------------
更新后id为4的信息为：User{id='4', name='jianxw_update_4', age=12}
获取用户id为4的信息：User{id='4', name='jianxw_update_4', age=12}
--------------结束更新用户id为4的信息-------------
--------------开始删除用户id为4的信息-------------
删除用户id为4的成功
进入方法，不是从缓存中取到id为4的值
获取用户id为4的信息：User{id='4', name='jianxw_4', age=12}
--------------结束删除用户id为4的信息-------------

```

通过结果可以看到，查询的时候缓存已经起作用，同时更新缓存过后，重新获取数据还是从缓存中取，而且更新的数据已经成功更新，删除缓存过后重新获取的时候需要重新进行缓存。

####项目文件目录结构

项目的目录结构为：
![](/img/20161212/spring_boot_redis.png)

####总结

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;文章主要介绍redis简单的操作以及对象的缓存操作，redis集群配置优化这些没有进行介绍，如果感兴趣的可以参照文档：https://redis.io/topics/cluster-tutorial。