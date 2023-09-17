---
title: "Spring整合kafka"
date: 2023-09-17T23:00:00+08:00
tags: ["java"]
draft: false
---

### 介绍
消息队列

### Kafka安装
拉取镜像
```
docker pull spotify/kafka
```
运行容器
```
docker run -d --name kafka-server -p 2181:2181 -p 9092:9092 -e ADVERTISED_HOST=127.0.0.1 -e ADVERTISED_PORT=9092 spotify/kafka
```

### Spring整合
#### Spring配置
引入依赖
```
<dependency>
    <groupId>org.springframework.kafka</groupId>
    <artifactId>spring-kafka</artifactId>
    <version>${spring-boot.version}</version>
</dependency>

<!-- kafka序列化 -->
<dependency>
    <groupId>com.fasterxml.jackson.core</groupId>
    <artifactId>jackson-databind</artifactId>
    <version>${jackson.version}</version>
</dependency>
```
application配置
```
spring:
  kafka:
    bootstrap-servers: localhost:9092
    producer:
      value-serializer: org.springframework.kafka.support.serializer.JsonSerializer
    consumer:
      value-deserializer: org.apache.kafka.common.serialization.ByteArrayDeserializer
```
Topic和消息转换器配置
```
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
```
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
```
@Service
public class KafkaConsumerService {
    // 接收消息
    @KafkaListener(id = "fooGroup", topics = "topic1")
    public void listen(Foo2 foo) {
        // TODO
    }
}
```