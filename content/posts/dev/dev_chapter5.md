---
title: "Spring 整合 Mongodb"
date: 2023-09-20T10:00:00+08:00
tags: ["java"]
draft: false
---

## 介绍
非关系型数据库

## 环境配置
拉取镜像
```
docker pull mongo
```
运行容器
```
docker run -d --name mongodb-server -p 27017:27017 mongo --auth
```
设置初始化用户
```
docker exec -it mongodb-server mongosh admin
> db.createUser({ user: 'admin', pwd: 'password', roles: [ { role: "userAdminAnyDatabase", db: "admin" } ] });
```
创建数据库及赋予权限
```
docker exec -it mongodb-server mongosh admin
> db.auth('admin', 'password');
> use testdb;
> db.createUser({ user: 'user', pwd: 'password', roles: [ { role: "dbOwner", db: "testdb" } ] });
```

## 示例
### Spring 配置
引入依赖
```
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
```
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
```
public interface BookRepository extends MongoRepository<Book, Integer> {
}
```
### 测试
```
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