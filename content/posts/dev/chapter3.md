---
title: "Spring 整合 Kafka"
date: 2023-09-17T23:00:00+08:00
categories: ["development"]
tags: ["java"]
draft: false
---

## 介绍

消息队列

## 环境配置

参考 [消息队列安装](../../ops/chapter4/#1-kafka)

## 示例

### Spring 配置

引入依赖
```xml
<dependency>
    <groupId>org.springframework.kafka</groupId>
    <artifactId>spring-kafka</artifactId>
    <version>${spring-boot.version}</version>
</dependency>

<!-- kafka 序列化 -->
<dependency>
    <groupId>com.fasterxml.jackson.core</groupId>
    <artifactId>jackson-databind</artifactId>
    <version>${jackson.version}</version>
</dependency>
```
application 配置
```yml
spring:
  kafka:
    bootstrap-servers: localhost:9092
    producer:
      value-serializer: org.springframework.kafka.support.serializer.JsonSerializer
    consumer:
      value-deserializer: org.apache.kafka.common.serialization.ByteArrayDeserializer
```
Topic 和消息转换器配置
```java
@Configuration
@EnableKafka
public class KafkaConfig {
    @Bean
    public NewTopic topic() {
        return new NewTopic("topic1", 1, (short) 1);
    }

    // 消息转换
    @Bean
    public RecordMessageConverter converter() {
        return new JsonMessageConverter();
    }
}
```
生产者配置
```java
@Service
public class KafkaProducerService {
    @Autowired
    private KafkaTemplate<String, Object> kafkaTemplate;

    // 发送消息
    public void sendFoo(String what) {
        kafkaTemplate.send("topic1", new Foo1(what));
    }
}
```
消费者配置
```java
@Service
public class KafkaConsumerService {
    // 接收消息
    @KafkaListener(id = "fooGroup", topics = "topic1")
    public void listen(Foo2 foo) {
        // TODO
    }
}
```

### 测试

```java
@SpringBootTest
class ApplicationTests {
    @Autowired
    private KafkaProducerService kafkaProducerService;

    @Test
    void sendMessageTest() {
        String what = "hello world!";
        kafkaProducerService.sendFoo(what);
    }
}
```