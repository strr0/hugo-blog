---
title: "Spring 整合 Kafka"
date: 2023-09-17T23:00:00+08:00
tags: ["java"]
draft: false
---

## 介绍
消息队列

## 环境配置
### 拉取镜像
```
docker pull bitnami/zookeeper
docker pull bitnami/kafka
```
### 运行容器
创建网络
```
docker network create app-tier --driver bridge
```
运行 zookeeper 容器
```
docker run -d --name zookeeper-server \
    -p 2181:2181 \
    --network app-tier \
    -e ALLOW_ANONYMOUS_LOGIN=yes \
    bitnami/zookeeper:latest
```
运行 kafka 容器
```
docker run -d --name kafka-server \
    -p 9092:9092 \
    --network app-tier \
    -e KAFKA_BROKER_ID=1 \
    -e KAFKA_CFG_LISTENERS=PLAINTEXT://:9092 \
    -e KAFKA_CFG_ADVERTISED_LISTENERS=PLAINTEXT://{ip}:9092 \
    -e KAFKA_CFG_ZOOKEEPER_CONNECT=zookeeper-server:2181 \
    -e ALLOW_PLAINTEXT_LISTENER=yes \
    bitnami/kafka:latest
```

## 示例
### Spring 配置
引入依赖
```
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
```
spring:
  kafka:
    bootstrap-servers: localhost:9092
    producer:
      value-serializer: org.springframework.kafka.support.serializer.JsonSerializer
    consumer:
      value-deserializer: org.apache.kafka.common.serialization.ByteArrayDeserializer
```
Topic 和消息转换器配置
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
### 测试
```
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