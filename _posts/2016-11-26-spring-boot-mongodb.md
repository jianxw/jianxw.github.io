---
layout: post
title: mongodb的安装配置在spring boot中的使用
tags:
- mongodb
- spring boot
categories: mongodb
---

　　MongoDB是一个基于分布式文件存储的nosql数据库，文章主要介绍spring boot中如何来连接和使用mongodb，同时对mongodb的安装配置，以及结合spring boot对mongodb具体的操作（增增删改查、模糊查询、根据位置查询、正则表达式查询、分页查询和排序等）进行总结和分享，希望对要在spring boot中使用mongodb来存储数据有一定的帮助。

<!-- more -->

#### 安装指南

- **安装和启动**

下载地址：https://www.mongodb.com/download-center?jmp=docs

解压和放到对应目录以及创建mongodb的数据存放目录：

```bash
[root@dev data]# tar -xzf mongodb-linux-x86_64-3.2.11.tgz
[root@dev data]# mv mongodb-linux-x86_64-3.2.11 /opt/ -rf
```

将mongodb加入到环境变量：

```bash
[root@dev data]# export PATH=$PATH:/opt/mongodb-linux-x86_64-3.2.11/bin
```

启动mongodb和连接mongo：

```bash
[root@dev data]# mongod --fork --logpath /var/log/mongodb.log
[root@dev data]# mongo
```

如果出现如下的内容表明连接成功

```bash
[root@dev /]# mongo
MongoDB shell version: 3.2.11
connecting to: test
Server has startup warnings:
2016-11-23T18:27:24.160+0800 I CONTROL  [initandlisten] ** WARNING: You are running this process as the root user, which is not recommended.
2016-11-23T18:27:24.160+0800 I CONTROL  [initandlisten]
2016-11-23T18:27:24.160+0800 I CONTROL  [initandlisten]
2016-11-23T18:27:24.160+0800 I CONTROL  [initandlisten] ** WARNING: /sys/kernel/mm/transparent_hugepage/enabled is 'always'.
2016-11-23T18:27:24.160+0800 I CONTROL  [initandlisten] **        We suggest setting it to 'never'
2016-11-23T18:27:24.160+0800 I CONTROL  [initandlisten]
2016-11-23T18:27:24.160+0800 I CONTROL  [initandlisten] ** WARNING: /sys/kernel/mm/transparent_hugepage/defrag is 'always'.
2016-11-23T18:27:24.160+0800 I CONTROL  [initandlisten] **        We suggest setting it to 'never'
2016-11-23T18:27:24.160+0800 I CONTROL  [initandlisten]
>
```

- **修改admin的密码和创建新的库和用户**

修改admin的密码：

```bash

[root@dev /]# mongo
MongoDB shell version: 3.2.11
connecting to: test
Server has startup warnings:
2016-11-23T18:27:24.160+0800 I CONTROL  [initandlisten] ** WARNING: You are running this process as the root user, which is not recommended.
2016-11-23T18:27:24.160+0800 I CONTROL  [initandlisten]
2016-11-23T18:27:24.160+0800 I CONTROL  [initandlisten]
2016-11-23T18:27:24.160+0800 I CONTROL  [initandlisten] ** WARNING: /sys/kernel/mm/transparent_hugepage/enabled is 'always'.
2016-11-23T18:27:24.160+0800 I CONTROL  [initandlisten] **        We suggest setting it to 'never'
2016-11-23T18:27:24.160+0800 I CONTROL  [initandlisten]
2016-11-23T18:27:24.160+0800 I CONTROL  [initandlisten] ** WARNING: /sys/kernel/mm/transparent_hugepage/defrag is 'always'.
2016-11-23T18:27:24.160+0800 I CONTROL  [initandlisten] **        We suggest setting it to 'never'
2016-11-23T18:27:24.160+0800 I CONTROL  [initandlisten]
>use admin;
>db.createUser(
  {
    user: "admin",
    pwd: "admin",
    roles: [ { role: "userAdminAnyDatabase", db: "admin" } ]
  }
);

```

创建新的数据库和用户名密码：

```bash
>use test_wjx;
>db.createUser(
  {
    user: "wjx",
    pwd: "123456",
    roles: [ { role: "readWrite", db: "test_wjx" } ]
  }
);
```

通过如下命令让用户和密码生效：

```bash
[root@dev /]# mongod --auth --port 27017 --dbpath /data/db
```

用对应的用户名和密码登录：

```bash
[root@dev /]# mongo --port 27017 -u wjx -p 123456 --authenticationDatabase test_wjx
```

#### spring boot中使用mongodb

项目的目录结构：

![](/img/20161127/spring_boot_mongodb.png)

通过maven引入需要的jar包：

```java
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.wjx</groupId>
    <artifactId>spring-boot-mongodb</artifactId>
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
            <artifactId>spring-boot-starter-data-mongodb</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
    </dependencies>

</project>
```

配置文件application.properties，增加mongodb的地址配置，连接到新建的mongodb的库test_wjx：

```java
spring.data.mongodb.uri=mongodb://wjx:123456@192.168.2.60:27017/test_wjx
```

需要保存的User类的代码为：

```java
package com.jianxw.mongo;

import org.springframework.data.annotation.Id;
import org.springframework.data.geo.Point;

import java.io.Serializable;

/**
 * @author jianxw
 * @date 2016/11/27
 */
public class User implements Serializable {
    private static final long serialVersionUID = 1L;

    @Id
    private String id;

    private String name;
    private int age;

    private Point location;

    public User() {
    }

    public User(String name, int age) {
        this.name = name;
        this.age = age;
    }

    public User(String name, int age, Point location) {
        this.name = name;
        this.age = age;
        this.location = location;
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

    public Point getLocation() {
        return location;
    }

    public void setLocation(Point location) {
        this.location = location;
    }

    @Override
    public String toString() {
        return String.format("用户：id=%s, name='%s', age='%s', location='%s'", id,
                name, age, location);
    }


}

```

创建mongodb访问的UserRepository接口，代码为：

```java
package com.jianxw.mongo;

import org.springframework.data.domain.Page;
import org.springframework.data.domain.Pageable;
import org.springframework.data.geo.Circle;
import org.springframework.data.geo.Point;
import org.springframework.data.mongodb.repository.MongoRepository;
import org.springframework.data.mongodb.repository.Query;

import java.util.List;

/**
 * @author jianxw
 * @date 2016/11/28 09:53
 */
public interface UserRepository extends MongoRepository<User, String> {

    User findByName(String name);

    List<User> findByAgeOrderByNameAsc(int age);

    List<User> findUserDistinctByName(String name);

    List<User> findByNameIgnoreCase(String name);

    Page<User> findFirst5ByName(String name, Pageable pageable);

    List<User> findByAge(int age);

    List<User> findByAgeLessThan(int age);

    List<User> findByAgeBetween(int start, int end);

    List<User> findByNameNotNull();

    List<User> findByNameLike(String name);

    List<User> findByNameNot(String name);

    List<User> findByLocationNear(Point point);

    List<User> findByLocationWithin(Circle circle);

    /**
     * $options  为i 表示不区分大小写
     * fields 制定只返回哪些字段的值
     *
     * @param name
     * @param ageFrom
     * @param ageTo
     * @param page
     * @return
     */
    @Query(value = "{ 'name':{'$regex':?0,'$options':'i'}, 'age': {'$gte':?1,'$lte':?2}}", fields = "{ 'name' : 1}")
    Page<User> findByNameAndAgeRange(String name, double ageFrom, double ageTo, Pageable page);
}

```

建立SpringBoot启动类，初始化连接mongodb的容器，同时调用数据访问层对数据库进行增删改查（包含模糊查询、位置查询、范围查询等）：

```java

package com.jianxw.mongo;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.CommandLineRunner;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.data.domain.PageRequest;
import org.springframework.data.domain.Pageable;
import org.springframework.data.geo.Circle;
import org.springframework.data.geo.Point;
import org.springframework.data.mongodb.core.MongoTemplate;
import org.springframework.data.mongodb.core.query.Query;
import org.springframework.data.mongodb.core.query.Update;

import static org.springframework.data.mongodb.core.query.Criteria.where;

@SpringBootApplication
public class SampleMongoApplication implements CommandLineRunner {

    @Autowired
    private UserRepository repository;

    @Autowired
    private MongoTemplate mongoTemplate;

    @Override
    public void run(String... args) throws Exception {

        //删除所有
        repository.deleteAll();
        //插入数据
        mongoTemplate.insert(new User("test2", 30));
        User saveUser = repository.save(new User("wjx", 20));
        repository.save(new User("test", 10));
        repository.save(new User("test", 33));
        repository.save(new User("test", 1));
        repository.save(new User("test", 2));
        repository.save(new User("test", 3));
        repository.save(new User("test", 4));
        repository.save(new User("test", 20));
        repository.save(new User("abc", 20));
        repository.save(new User("wjx", 20));
        repository.save(new User("TEST", 44));
        repository.save(new User("test1", 15));
        repository.save(new User("test_point", 25, new Point(-73.99279, 40.719296)));
        repository.save(new User("test_point1", 66, new Point(-73.99279, 40.719296)));
        repository.save(new User(15));
        //更新数据
        mongoTemplate.updateFirst(new Query(where("id").is(saveUser.getId())),
                new Update().set("age", 50),
                User.class);

        //查找所有
        System.out.println("查询所有User findAll():");
        for (User user : repository.findAll()) {
            System.out.println(user);
        }
        //根据名字查询 并且升序排列
        System.out.println("根据名字查询,并且升序排列 findByAgeOrderByNameAsc:");
        for (User user : repository.findByAgeOrderByNameAsc(20)) {
            System.out.println(user);
        }

        //根据名字查询 distinct
        System.out.println("根据名字查询distinct, findUserDistinctByName:");
        for (User user : repository.findUserDistinctByName("test")) {
            System.out.println(user);
        }

        //根据名字查询 不区分大小写
        System.out.println("根据名字查询，不区分大小写 findByNameIgnoreCase:");
        for (User user : repository.findByNameIgnoreCase("test")) {
            System.out.println(user);
        }

        //根据名字查询 分页查询
        System.out.println("根据名字查询，分页查询 findFirst5ByName:");
        Pageable pageable1 = new PageRequest(0, 5);
        for (User user : repository.findFirst5ByName("test", pageable1)) {
            System.out.println(user);
        }

        //根据名字查询
        System.out.println("根据名字查询 findByName('wjx'):");
        System.out.println(repository.findByName("wjx"));
        //根据年龄查询 等于
        System.out.println("根据年龄查询 findByAge(30):");
        for (User customer : repository.findByAge(30)) {
            System.out.println(customer);
        }

        //根据年龄查询 小于
        System.out.println("根据年龄查询 findByAgeLessThan(30):");
        for (User customer : repository.findByAgeLessThan(30)) {
            System.out.println(customer);
        }
        //根据年龄查询 范围
        System.out.println("根据年龄查询 范围 findByAgeBetween(10, 20):");
        for (User customer : repository.findByAgeBetween(9, 20)) {
            System.out.println(customer);
        }
        //根据姓名查询 姓名不为空
        System.out.println("根据姓名查询 姓名不为空 findByNameNotNull():");
        for (User customer : repository.findByNameNotNull()) {
            System.out.println(customer);
        }
        //根据姓名查询 模糊查询
        System.out.println("根据姓名查询 模糊查询 findByNameLike(\"test\"):");
        for (User customer : repository.findByNameLike("test")) {
            System.out.println(customer);
        }
        //根据姓名查询 不等于
        System.out.println("根据姓名查询 不等于 findByNameNot(\"test\"):");
        for (User customer : repository.findByNameNot("test")) {
            System.out.println(customer);
        }
        //根据地理位置查询
        System.out.println("根据地理位置查询 findByLocationNear:");
        for (User customer : repository.findByLocationNear(new Point(-73.99279, 40.719296))) {
            System.out.println(customer);
        }
        //根据地理位置查询 范围
        System.out.println("根据地理位置查询 findByLocationWithin:");
        for (User customer : repository.findByLocationWithin(new Circle(new Point(-73.99279, 40.719296), 1000))) {
            System.out.println(customer);
        }
        //名字和年龄的范围查询
        System.out.println("名字和年龄的范围查询 findByNameAndAgeRange:");
        Pageable pageable = new PageRequest(0, 10);
        for (User customer : repository.findByNameAndAgeRange("test", 9, 20, pageable)) {
            System.out.println(customer);
        }
    }

    public static void main(String[] args) throws Exception {
        SpringApplication.run(SampleMongoApplication.class, args);
    }

}

```

由于代码中要进行根据地理位置的匹配，需要增加根据坐标查询需要的mongodb数据库的"2d"索引：

```bash
> db.user.createIndex( { location : "2d" } );

> db.user.getIndexes();
[
        {
                "v" : 1,
                "key" : {
                        "_id" : 1
                },
                "name" : "_id_",
                "ns" : "test_wjx.user"
        },
        {
                "v" : 1,
                "key" : {
                        "location" : "2d"
                },
                "name" : "location_2d",
                "ns" : "test_wjx.user"
        }
]
```

#### 总结

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;spring-data-mongodb的MongoRepository暂时不支持对数据进行更新操作，所以用的MongoTemplate.update进行数据的更新。文章主要是对mongodb的使用进行了介绍，比如增删改查、模糊查询、范围查询、正则表达式查询、分页查询排序等。