---
title: "Spring 整合 Redis"
date: 2023-11-30T10:00:00+08:00
tags: ["java"]
draft: false
---

## 介绍  
缓存技术

## 1 环境配置  
拉取镜像
```
docker pull redis
```
运行容器
```
docker run -d --name redis-server -p 6379:6379 redis --requirepass password（密码）
```

## 2 使用

### 2.1 基于 Spring data api

#### 2.1.1 Spring 配置
引入依赖
```
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
```
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
```
@Service
public class BookService {
    @Resource
    private RedisTemplate<String, Book> redisTemplate;
    // TODO
}
```

#### 2.1.2 RedisTemplate 操作 redis  
设置 key value
```
public void setBook(String key, Book book) {
    redisTemplate.opsForValue().set(key, book);
}
```
获取 value
```
public Book getBook(String key) {
    return redisTemplate.opsForValue().get(key);
}
```
删除 key
```
public Boolean removeBook(String key) {
    return redisTemplate.delete(key);
}
```

### 2.2 基于 Jedis 客户端

#### 2.2.1 建立连接及连接测试
```
    Jedis jedis = new Jedis("localhost", 6379);
    System.out.println(jedis.ping());
    jedis.close();
```

#### 2.2.2 其他操作  
设置 key value
```
    jedis.set("key", "value");
```
获取 value
```
    jedis.get("key");
```
设置过期时间
```
    jedis.expire("key", 100);
```
删除 key
```
    jedis.del("key");
```

### 2.3 基于 Redisson 客户端

#### 2.3.1 Spring 配置  
引入依赖
```
<dependency>
    <groupId>org.redisson</groupId>
    <artifactId>redisson-spring-boot-starter</artifactId>
    <version>${redisson.version}</version>
</dependency>
```
application 配置
```
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

#### 2.3.2 RedissonClient 操作 redis  
设置 key value
```
    RBucket<Integer> bucket = redissonClient.getBucket("key");
    bucket.set(100);
```
获取 value
```
    RBucket<Integer> bucket = redissonClient.getBucket("key");
    System.out.println(bucket.get());
```
删除 key
```
    RBucket<Integer> bucket = redissonClient.getBucket("key");
    bucket.delete();
```
获取 list
```
    RList<Object> rList = redissonClient.getList("key");
    List<Object> dataList = rList.readAll();
```
添加 list 元素
```
    RList<Object> rList = redissonClient.getList("key");
    rList.add("something");
```
清空 list
```
    RList<Object> rList = redissonClient.getList("key");
    rList.clear();
```