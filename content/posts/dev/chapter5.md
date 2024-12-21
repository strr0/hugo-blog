---
title: "Spring 整合 MongoDB"
date: 2023-09-20T10:00:00+08:00
categories: ["development"]
tags: ["java"]
draft: false
---

## 介绍

非关系型数据库

## 环境配置

参考 [MongoDB 安装](../../ops/chapter5)

## 示例

### Spring 配置

引入依赖
```xml
<dependency>
    <groupId>org.springframework.data</groupId>
    <artifactId>spring-data-mongodb</artifactId>
    <version>${mongo.version}</version>
</dependency>

<dependency>
    <groupId>org.mongodb</groupId>
    <artifactId>mongodb-driver-sync</artifactId>
    <version>${mongo-driver.version}</version>
</dependency>
```
application 配置
```yml
spring:
  data:
    mongodb:
      authentication-database: testdb
      database: testdb
      host: localhost
      port: 27017
      username: user
      password: password
```
repository 配置
```java
public interface BookRepository extends MongoRepository<Book, Integer> {
}
```

### 测试

```java
@SpringBootTest
class ApplicationTests {
    @Autowired
    private BookRepository bookRepository;

    @Test
    void addBookTest() {
        Book book = new Book();
        book.setId(1);
        book.setName("Spring in action");
        book.setAuthor("xxx");
        bookRepository.save(book);
        System.out.println("save book success.");
    }

    @Test
    void getBookTest() {
        bookRepository.findById(1).map(book -> {
            System.out.println("get " + book);
            return book;
        });
    }
}
```