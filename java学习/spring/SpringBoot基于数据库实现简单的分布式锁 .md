# SpringBoot基于数据库实现简单的分布式锁

## 1. 简介

分布式锁的方式有很多种，通常方案有： 

- 基于mysql数据库 
- 基于redis 
- 基于ZooKeeper 

网上的实现方式有很多，本文主要介绍的是如果使用mysql实现简单的分布式锁，加锁流程如下图： 

![title](https://raw.githubusercontent.com/lllpla/img/master/gitnote/2020/04/10/1586510441534-1586510441595.png)

其实大致思想如下： 

1. 根据一个值来获取锁（也就是我这里的tag），如果当前不存在锁，那么在数据库插入一条记录，然后进行处理业务，当结束，释放锁（删除锁）。
2. 如果存在锁，判断锁是否过期，如果过期则更新锁的有效期，然后继续处理业务，当结束时，释放锁。如果没有过期，那么获取锁失败，退出。 

## 2.数据库设计

数据库表是由JPA自动生成的，稍后会对实体进行介绍，内容如下： 

```mysql
CREATE TABLE `lock_info` ( 
 `id` bigint(20) NOT NULL, 
 `expiration_time` datetime NOT NULL, 
 `status` int(11) NOT NULL, 
 `tag` varchar(255) NOT NULL, 
 PRIMARY KEY (`id`), 
 UNIQUE KEY `uk_tag` (`tag`) 
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci; 
```

其中： 

- id：主键 
- tag：锁的标示，以订单为例，可以锁订单id 
- expiration_time：过期时间 
- status：锁状态，0，未锁，1，已经上锁 

## 3.实现

新建项目，在项目中加入jpa和mysql依赖，完整内容如下： 

```xml
<?xml version="1.0" encoding="UTF-8"?> 
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" 
xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd"> 
<modelVersion>4.0.0</modelVersion> 
<parent> 
<groupId>org.springframework.boot</groupId> 
<artifactId>spring-boot-starter-parent</artifactId> 
<version>2.0.3.RELEASE</version> 
<relativePath/> <!-- lookup parent from repository --> 
</parent> 
<groupId>com.dalaoyang</groupId> 
<artifactId>springboot2_distributed_lock_mysql</artifactId> 
<version>0.0.1-SNAPSHOT</version> 
<name>springboot2_distributed_lock_mysql</name> 
<description>springboot2_distributed_lock_mysql</description> 
 
<properties> 
<java.version>1.8</java.version> 
</properties> 
 
<dependencies> 
<dependency> 
<groupId>org.springframework.boot</groupId> 
<artifactId>spring-boot-starter-web</artifactId> 
</dependency> 
<dependency> 
<groupId>org.springframework.boot</groupId> 
<artifactId>spring-boot-starter-data-jpa</artifactId> 
</dependency> 
 
<dependency> 
<groupId>mysql</groupId> 
<artifactId>mysql-connector-java</artifactId> 
<scope>runtime</scope> 
</dependency> 
<dependency> 
<groupId>org.springframework.boot</groupId> 
<artifactId>spring-boot-starter-test</artifactId> 
<scope>test</scope> 
</dependency> 
 
<dependency> 
<groupId>org.projectlombok</groupId> 
<artifactId>lombok</artifactId> 
<version>1.16.22</version> 
<scope>provided</scope> 
</dependency> 
</dependencies> 
 
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

 