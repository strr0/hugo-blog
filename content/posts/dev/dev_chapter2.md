---
title: "Spring 整合 Redis"
date: 2023-09-14T23:00:00+08:00
tags: ["java"]
draft: false
---

### 介绍
缓存技术

### Redis 安装
拉取镜像
```
docker pull redis
```
运行容器
```
docker run -d --name redis-server -p 6379:6379 redis --requirepass password（密码）
```

### Spring 整合
#### Spring 配置
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
#### RedisTemplate 操作 redis
设置key value
```
public void setBook(String key, Book book) {
    redisTemplate.opsForValue().set(key, book);
}
```
获取value
```
public Book getBook(String key) {
    return redisTemplate.opsForValue().get(key);
}
```
删除key
```
public Boolean removeBook(String key) {
    return redisTemplate.delete(key);
}
```