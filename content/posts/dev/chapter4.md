---
title: "Spring 整合 RabbitMQ"
date: 2023-09-19T15:00:00+08:00
categories: ["development"]
tags: ["java"]
draft: false
---

## 介绍

消息队列

## 环境配置

参考 [消息队列安装](../../ops/chapter4/#2-rabbitmq)

## 示例

### 依赖配置

引入依赖
```
<dependency>
    <groupId>org.springframework.amqp</groupId>
    <artifactId>spring-rabbit</artifactId>
    <version>${amqp.version}</version>
</dependency>

<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-messaging</artifactId>
</dependency>
```
application 配置
```
spring:
  rabbitmq:
    host: localhost
    port: 5672
    username: user
    password: password
```

### 直连模式（Direct）

队列及交换机配置
```
@Configuration
public class RabbitDirectConfig {
    public final static String DIRECTNAME = "strr-direct";

    @Bean
    protected Queue queue() {
        return new Queue("hello-queue");
    }

    @Bean
    protected DirectExchange directExchange() {
        return new DirectExchange(DIRECTNAME, true, false);
    }

    @Bean
    protected Binding binding() {
        return BindingBuilder.bind(queue()).to(directExchange()).with("direct");
    }
}
```
生产者配置
```
@Service
public class ProducerService {
    @Autowired
    private RabbitTemplate rabbitTemplate;

    public void sendDirect(String key, String what) {
        rabbitTemplate.convertAndSend(key, new Foo(what));
    }
}
```
消费者配置
```
@Service
public class DirectConsumerService {
    @RabbitListener(queues = "hello-queue")
    public void listen(Foo foo) {
        // TODO
    }
}
```
测试
```
@SpringBootTest
class ApplicationTests {
    @Autowired
    private ProducerService producerService;

    @Test
    void sendDirectTest() {
        producerService.sendDirect("hello-queue", "hello world!");
    }
}
```

### 广播模式（Fanout）

队列及交换机配置
```
@Configuration
public class RabbitFanoutConfig {
    public final static String FANOUTNAME = "strr-fanout";

    @Bean
    protected FanoutExchange fanoutExchange() {
        return new FanoutExchange(FANOUTNAME, true, false);
    }

    @Bean
    protected Queue queueOne() {
        return new Queue("queue-one");
    }

    @Bean
    protected Queue queueTwo() {
        return new Queue("queue-two");
    }

    @Bean
    protected Binding bindingOne() {
        return BindingBuilder.bind(queueOne()).to(fanoutExchange());
    }

    @Bean
    protected Binding bindingTwo() {
        return BindingBuilder.bind(queueTwo()).to(fanoutExchange());
    }
}
```
生产者配置
```
@Service
public class ProducerService {
    @Autowired
    private RabbitTemplate rabbitTemplate;

    public void sendFanout(String what) {
        rabbitTemplate.convertAndSend(RabbitFanoutConfig.FANOUTNAME, null, new Foo(what));
    }
}
```
消费者配置
```
@Service
public class FanoutConsumerService {
    @RabbitListener(queues = "queue-one")
    public void listen1(Foo foo) {
        // TODO
    }

    @RabbitListener(queues = "queue-two")
    public void listen2(Foo foo) {
        // TODO
    }
}
```
测试
```
@SpringBootTest
class ApplicationTests {
    @Autowired
    private ProducerService producerService;

    @Test
    void sendFanoutTest() {
        producerService.sendFanout("hello fanout!");
    }
}
```

### Topic模式（根据路由key模糊匹配）

队列及交换机配置
```
@Configuration
public class RabbitTopicConfig {
    public final static String TOPICNAME = "strr-topic";

    @Bean
    protected TopicExchange topicExchange() {
        return new TopicExchange(TOPICNAME, true, false);
    }

    @Bean
    protected Queue queue1() {
        return new Queue("topic1-queue");
    }

    @Bean
    protected Queue queue2() {
        return new Queue("topic2-queue");
    }

    @Bean
    protected Queue queue3() {
        return new Queue("topic3-queue");
    }

    @Bean
    protected Binding queue1Binding() {
        return BindingBuilder.bind(queue1()).to(topicExchange())
                .with("topic1.#");
    }

    @Bean
    protected Binding queue2Binding() {
        return BindingBuilder.bind(queue2()).to(topicExchange())
                .with("topic2.#");
    }

    @Bean
    protected Binding queue3Binding() {
        return BindingBuilder.bind(queue3()).to(topicExchange())
                .with("topic3.#");
    }
}
```
生产者配置
```
@Service
public class ProducerService {
    @Autowired
    private RabbitTemplate rabbitTemplate;

    public void sendTopic(String key, String what) {
        rabbitTemplate.convertAndSend(RabbitTopicConfig.TOPICNAME, key, new Foo(what));
    }
}
```
消费者配置
```
@Service
public class TopicConsumerService {
    @RabbitListener(queues = "topic1-queue")
    public void listen1(Foo foo) {
        // TODO
    }

    @RabbitListener(queues = "topic2-queue")
    public void listen2(Foo foo) {
        // TODO
    }

    @RabbitListener(queues = "topic3-queue")
    public void listen3(Foo foo) {
        // TODO
    }
}
```
测试
```
@SpringBootTest
class ApplicationTests {
    @Autowired
    private ProducerService producerService;

    @Test
    void sendTopicTest() {
        producerService.sendTopic("topic1.aa", "hello topic1!");
        producerService.sendTopic("topic2.bb", "hello topic2!");
        producerService.sendTopic("topic3.cc", "hello topic3!");
    }
}
```

### Header模式（根据header匹配）

队列及交换机配置
```
@Configuration
public class RabbitHeaderConfig {
    public final static String HEADERNAME = "strr-header";

    @Bean
    protected HeadersExchange headersExchange() {
        return new HeadersExchange(HEADERNAME, true, false);
    }

    @Bean
    protected Queue queueName() {
        return new Queue("name-queue");
    }

    @Bean
    protected Queue queueAge() {
        return new Queue("age-queue");
    }

    @Bean
    protected Binding bindingName() {
        Map<String, Object> map = new HashMap<>();
        map.put("name", "strr");
        return BindingBuilder.bind(queueName())
                .to(headersExchange()).whereAny(map).match();
    }

    @Bean
    protected Binding bindingAge() {
        return BindingBuilder.bind(queueAge())
                .to(headersExchange()).where("age").exists();
    }
}
```
生产者配置
```
@Service
public class ProducerService {
    @Autowired
    private RabbitTemplate rabbitTemplate;

    public void sendHeader(Message message) {
        rabbitTemplate.convertAndSend(RabbitHeaderConfig.HEADERNAME, null, message);
    }
}
```
消费者配置
```
@Service
public class HeaderConsumerService {
    @RabbitListener(queues = "name-queue")
    public void handler1(byte[] msg) {
        // TODO
    }

    @RabbitListener(queues = "age-queue")
    public void handler2(byte[] msg) {
        // TODO
    }
}
```
测试
```
@SpringBootTest
class ApplicationTests {
    @Autowired
    private ProducerService producerService;

    @Test
    void sendHeaderTest() {
        Message nameMsg = MessageBuilder.withBody("hello header! name-queue".getBytes())
                .setHeader("name", "strr").build();
        Message ageMsg = MessageBuilder.withBody("hello header! age-queue".getBytes())
                .setHeader("age", "99").build();
        producerService.sendHeader(nameMsg);
        producerService.sendHeader(ageMsg);
    }
}
```