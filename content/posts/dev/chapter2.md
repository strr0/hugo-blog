---
title: "Spring 整合 Redis"
date: 2023-11-30T10:00:00+08:00
categories: ["development"]
tags: ["java"]
draft: false
---

## 介绍

缓存技术

## 1 环境配置

参考 [Redis 安装](../../ops/chapter3)

## 2 示例

### 2.1 基于 Spring Data

#### 2.1.1 Spring 配置

引入依赖
```xml
<dependency>
    <groupId>org.springframework.data</groupId>
    <artifactId>spring-data-redis</artifactId>
    <version>${spring-boot.version}</version>
</dependency>

<dependency>
    <groupId>redis.clients</groupId>
    <artifactId>jedis</artifactId>
</dependency>
```
application 配置
```yml
spring:
  redis:
    database: 0
    host: localhost
    port: 6379
    password: password
    jedis:
      pool:
        max-active: 8
        max-idle: 8
        max-wait: -1ms
        min-idle: 0
```
RedisTemplate 注入 BookService
```java
@Service
public class BookService {
    @Resource
    private RedisTemplate<String, Book> redisTemplate;
    // TODO
}
```

#### 2.1.2 RedisTemplate 操作 Redis

设置 key value
```java
public void setBook(String key, Book book) {
    redisTemplate.opsForValue().set(key, book);
}
```
获取 value
```java
public Book getBook(String key) {
    return redisTemplate.opsForValue().get(key);
}
```
删除 key
```java
public Boolean removeBook(String key) {
    return redisTemplate.delete(key);
}
```

### 2.2 基于 Jedis 客户端

#### 2.2.1 建立连接及连接测试

```java
Jedis jedis = new Jedis("localhost", 6379);
System.out.println(jedis.ping());
jedis.close();
```

#### 2.2.2 其他操作

设置 key value
```java
jedis.set("key", "value");
```
获取 value
```java
jedis.get("key");
```
设置过期时间
```java
jedis.expire("key", 100);
```
删除 key
```java
jedis.del("key");
```

### 2.3 基于 Redisson 客户端

#### 2.3.1 Spring 配置

引入依赖
```xml
<dependency>
    <groupId>org.redisson</groupId>
    <artifactId>redisson-spring-boot-starter</artifactId>
    <version>${redisson.version}</version>
</dependency>
```
application 配置
```yml
spring:
  data:
    redis:
      host: localhost
      port: 6379
      database: 0
      password: password
      timeout: 30s
      ssl.enabled: false
```

#### 2.3.2 RedissonClient 操作 Redis

设置 key value
```java
RBucket<Integer> bucket = redissonClient.getBucket("key");
bucket.set(100);
```
获取 value
```java
RBucket<Integer> bucket = redissonClient.getBucket("key");
System.out.println(bucket.get());
```
删除 key
```java
RBucket<Integer> bucket = redissonClient.getBucket("key");
bucket.delete();
```
获取 list
```java
RList<Object> rList = redissonClient.getList("key");
List<Object> dataList = rList.readAll();
```
添加 list 元素
```java
RList<Object> rList = redissonClient.getList("key");
rList.add("something");
```
清空 list
```java
RList<Object> rList = redissonClient.getList("key");
rList.clear();
```