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

### 3.1 POM

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

### 3.2 配置文件

配置文件配置了一下数据库信息和jpa的基本配置，如下： 

```properties
server.port=20001 
 
##数据库配置 
##数据库地址 
spring.datasource.url=jdbc:mysql://localhost:3306/lock?characterEncoding=utf8&useSSL=false 
##数据库用户名 
spring.datasource.username=root 
##数据库密码 
spring.datasource.password=12345678 
##数据库驱动 
spring.datasource.driver-class-name=com.mysql.jdbc.Driver 
 
##validate 加载hibernate时，验证创建数据库表结构 
##create  每次加载hibernate，重新创建数据库表结构，这就是导致数据库表数据丢失的原因。 
##create-drop    加载hibernate时创建，退出是删除表结构 
##update         加载hibernate自动更新数据库结构 
##validate 启动时验证表的结构，不会创建表 
##none 启动时不做任何操作 
spring.jpa.hibernate.ddl-auto=update 
 
##控制台打印sql 
spring.jpa.show-sql=true 
##设置innodb 
spring.jpa.database-platform=org.hibernate.dialect.MySQL5InnoDBDialect 
```



### 3.3 实体

实体类如下，这里给tag字段设置了唯一索引，防止重复插入相同的数据： 

```java
package com.dalaoyang.entity; 
 
import lombok.Data; 
import javax.persistence.*; 
import java.util.Date; 
 
@Data 
@Entity 
@Table(name = "LockInfo", 
    uniqueConstraints={@UniqueConstraint(columnNames={"tag"},name = "uk_tag")}) 
public class Lock { 
 
  public final static Integer LOCKED_STATUS = 1; 
  public final static Integer UNLOCKED_STATUS = 0; 
 
  /** 
   * 主键id 
   */ 
  @Id 
  @GeneratedValue(strategy = GenerationType.AUTO) 
  private Long id; 
 
  /** 
   * 锁的标示，以订单为例，可以锁订单id 
   */ 
  @Column(nullable = false) 
  private String tag; 
 
  /** 
   * 过期时间 
   */ 
  @Column(nullable = false) 
  private Date expirationTime; 
 
  /** 
   * 锁状态，0，未锁，1，已经上锁 
   */ 
  @Column(nullable = false) 
  private Integer status; 
 
  public Lock(String tag, Date expirationTime, Integer status) { 
    this.tag = tag; 
    this.expirationTime = expirationTime; 
    this.status = status; 
  } 
 
  public Lock() { 
  } 
} 
```

### 3.4 repository 

repository层只添加了两个简单的方法，根据tag查找锁和根据tag删除锁的操作，内容如下： 

```java
package com.dalaoyang.repository; 

import com.dalaoyang.entity.Lock; 
import org.springframework.data.jpa.repository.JpaRepository; 


public interface LockRepository extends JpaRepository<Lock, Long> { 

  Lock findByTag(String tag); 

  void deleteByTag(String tag); 
} 
```

### 3.5 service 

service接口定义了两个方法，获取锁和释放锁，内容如下： 

```java
package com.dalaoyang.service; 
 
 
public interface LockService { 
 
  /** 
   * 尝试获取锁 
   * @param tag 锁的键 
   * @param expiredSeconds 锁的过期时间（单位：秒），默认10s 
   * @return 
   */ 
  boolean tryLock(String tag, Integer expiredSeconds); 
 
  /** 
   * 释放锁 
   * @param tag 锁的键 
   */ 
  void unlock(String tag); 
} 
```

 

实现类对上面方法进行了实现，其内容与上述流程图中一致，这里不在做介绍，完整内容如下： 

```java
package com.dalaoyang.service.impl; 

import com.dalaoyang.entity.Lock; 
import com.dalaoyang.repository.LockRepository; 
import com.dalaoyang.service.LockService; 
import org.springframework.beans.factory.annotation.Autowired; 
import org.springframework.stereotype.Service; 
import org.springframework.transaction.annotation.Propagation; 
import org.springframework.transaction.annotation.Transactional; 
import org.springframework.util.StringUtils; 

import java.util.Calendar; 
import java.util.Date; 
import java.util.Objects; 


@Service 
public class LockServiceImpl implements LockService { 

  private final Integer DEFAULT_EXPIRED_SECONDS = 10; 

  @Autowired 
  private LockRepository lockRepository; 

  @Override 
  @Transactional(rollbackFor = Throwable.class) 
  public boolean tryLock(String tag, Integer expiredSeconds) { 
    if (StringUtils.isEmpty(tag)) { 
      throw new NullPointerException(); 
    } 
    Lock lock = lockRepository.findByTag(tag); 
    if (Objects.isNull(lock)) { 
      lockRepository.save(new Lock(tag, this.addSeconds(new Date(), expiredSeconds), Lock.LOCKED_STATUS)); 
      return true; 
    } else { 
      Date expiredTime = lock.getExpirationTime(); 
      Date now = new Date(); 
      if (expiredTime.before(now)) { 
        lock.setExpirationTime(this.addSeconds(now, expiredSeconds)); 
        lockRepository.save(lock); 
        return true; 
      } 
    } 
    return false; 
  } 

  @Override 
  @Transactional(rollbackFor = Throwable.class) 
  public void unlock(String tag) { 
    if (StringUtils.isEmpty(tag)) { 
      throw new NullPointerException(); 
    } 
    lockRepository.deleteByTag(tag); 
  } 

  private Date addSeconds(Date date, Integer seconds) { 
    if (Objects.isNull(seconds)){ 
      seconds = DEFAULT_EXPIRED_SECONDS; 
    } 
    Calendar calendar = Calendar.getInstance(); 
    calendar.setTime(date); 
    calendar.add(Calendar.SECOND, seconds); 
    return calendar.getTime(); 
  } 
} 
```

### 3.6 测试类 

创建了一个测试的controller进行测试，里面写了一个test方法，方法在获取锁的时候会sleep 2秒，便于我们进行测试。完整内容如下: 

```java
package com.dalaoyang.controller; 

import com.dalaoyang.service.LockService; 
import org.springframework.beans.factory.annotation.Autowired; 
import org.springframework.web.bind.annotation.GetMapping; 
import org.springframework.web.bind.annotation.RestController; 


@RestController 
public class TestController { 

  @Autowired 
  private LockService lockService; 

  @GetMapping("/tryLock") 
  public Boolean tryLock(String tag, Integer expiredSeconds) { 
    return lockService.tryLock(tag, expiredSeconds); 
  } 

  @GetMapping("/unlock") 
  public Boolean unlock(String tag) { 
    lockService.unlock(tag); 
    return true; 
  } 

  @GetMapping("/test") 
  public String test(String tag, Integer expiredSeconds) { 
    if (lockService.tryLock(tag, expiredSeconds)) { 
      try { 
        //do something 
        //这里使用睡眠两秒，方便观察获取不到锁的情况 
        Thread.sleep(2000); 
      } catch (Exception e) { 

      } finally { 
        lockService.unlock(tag); 
      } 
      return "获取锁成功，tag是：" + tag; 
    } 
    return "当前tag：" + tag + "已经存在锁，请稍后重试！"; 

  } 
} 
```

## 4.测试

项目使用maven打包，分别使用两个端口启动，分别是20000和20001。 

```java
java -jar springboot2_distributed_lock_mysql-0.0.1-SNAPSHOT.jar --server.port=20001 

java -jar springboot2_distributed_lock_mysql-0.0.1-SNAPSHOT.jar --server.port=20000 
```

分别访问两个端口的项目，如图所示，只有一个请求可以获取锁。 