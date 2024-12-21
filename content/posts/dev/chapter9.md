---
title: "Spring 整合 Nacos"
date: 2024-01-11T23:00:00+08:00
categories: ["development"]
tags: ["java", "mysql"]
draft: false
---

## 介绍

构建云原生应用的动态服务发现、配置管理和服务管理平台。

## 1 快速开始

### 1.1 MySql 环境

参考 [MySQL 安装](../../ops/chapter12/#4-mysql-环境)

导入 [nacos](https://raw.githubusercontent.com/alibaba/nacos/develop/distribution/conf/mysql-schema.sql) 数据

### 1.2 Nacos 环境

参考 [Nacos 安装](../../ops/chapter12/#5-nacos-环境)

## 2 示例

### 2.1 Spring Cloud 配置中心

引入依赖
```xml
<dependency>
    <groupId>com.alibaba.cloud</groupId>
    <artifactId>spring-cloud-starter-alibaba-nacos-config</artifactId>
</dependency>
```
application 配置
```yml
spring:
  application:
    # 应用名称
    name: provider-service
  profiles:
    # 环境配置
    active: dev
  cloud:
    nacos:
      # nacos 服务地址
      server-addr: localhost:8848
      config:
        # 配置组
        group: DEFAULT_GROUP
        namespace: ${spring.profiles.active}
  config:
    import:
      - optional:nacos:${spring.application.name}.yml
```

### 2.2 Spring Cloud 注册中心及服务发现

#### 2.2.1 服务提供方配置

引入依赖
```xml
<dependency>
    <groupId>com.alibaba.cloud</groupId>
    <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
</dependency>
```
application 配置
```yml
spring:
  application:
    # 应用名称
    name: provider-service
  profiles:
    # 环境配置
    active: dev
  cloud:
    nacos:
      # nacos 服务地址
      server-addr: localhost:8848
      discovery:
        # 注册组
        group: DEFAULT_GROUP
        namespace: ${spring.profiles.active}
```
定义服务接口
```java
@RestController
public class HelloController {
    @GetMapping("/hello")
    public String hello() {
        return "provider:hello!";
    }
}
```

#### 2.2.2 服务消费者配置

引入依赖
```xml
<dependency>
    <groupId>com.alibaba.cloud</groupId>
    <artifactId>spring-cloud-starter-alibaba-nacos-config</artifactId>
</dependency>

<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-loadbalancer</artifactId>
</dependency>
```
application 配置
```yml
spring:
  application:
    # 应用名称
    name: consumer-service
  profiles:
    # 环境配置
    active: dev
  cloud:
    nacos:
      # nacos 服务地址
      server-addr: localhost:8848
      discovery:
        # 注册组
        group: DEFAULT_GROUP
        namespace: ${spring.profiles.active}
```
RestTemplate 配置
```java
@Configuration
@EnableDiscoveryClient
public class RestTemplateConfig {
    @Bean
    @LoadBalanced
    public RestTemplate restTemplate() {
        return new RestTemplate();
    }
}
```
定义服务客户端
```java
@Component
public class ProviderClient {
    private final RestTemplate restTemplate;

    public ProviderClient(RestTemplate restTemplate) {
        this.restTemplate = restTemplate;
    }

    public String hello() {
        ResponseEntity<String> exchange = restTemplate.exchange("http://provider-service/hello", HttpMethod.GET, null, String.class);
        return exchange.getBody();
    }
}
```

> @LoadBalanced 注解的作用是使 RestTemplate 开启负载均衡功能，在 RestTemplate 的 bean 上添加 @LoadBalanced 注解后，Spring 会为 RestTemplate bean 注册一个 loadBalancedInterceptor 拦截器，这个拦截器可以替换请求地址中的服务逻辑名（即上文中的 provider-service），将其转换为具体的服务地址，实现负载均衡。

### 2.3 Dubbo 注册中心及服务发现

#### 2.3.1 定义基础服务

创建 dubbo 公共服务模块，并定义服务接口
```java
public interface HelloService {
    String hello();
}
```

#### 2.3.2 服务提供方配置

引入依赖
```xml
<dependency>
    <groupId>org.apache.dubbo</groupId>
    <artifactId>dubbo-spring-boot-starter</artifactId>
    <version>${dubbo-spring-boot.version}</version>
</dependency>
```
application 配置
```yml
spring:
  application:
    # 应用名称
    name: dubbo-provider-service
  profiles:
    # 环境配置
    active: dev

dubbo:
  application:
    logger: slf4j
    # 关闭qos端口避免单机多生产者端口冲突 如需使用自行开启
    qos-enable: false
    # 元数据中心 local 本地 remote 远程 这里使用远程便于其他服务获取
    metadataType: remote
    # 可选值 interface、instance、all，默认是 all，即接口级地址、应用级地址都注册
    register-mode: instance
    service-discovery:
      # FORCE_INTERFACE，只消费接口级地址，如无地址则报错，单订阅 2.x 地址
      # APPLICATION_FIRST，智能决策接口级/应用级地址，双订阅
      # FORCE_APPLICATION，只消费应用级地址，如无地址则报错，单订阅 3.x 地址
      migration: FORCE_APPLICATION
  protocol:
    # 使用 dubbo 协议通信
    name: dubbo
    # dubbo 协议端口(-1表示自增端口,从20880开始)
    port: -1
  # 注册中心配置
  registry:
    address: nacos://${spring.cloud.nacos.server-addr}
    group: DUBBO_GROUP
    parameters:
      namespace: ${spring.profiles.active}
  scan:
    # 接口实现类扫描
    base-packages: com.strr.provider.dubbo
```
定义 Dubbo 服务
```java
@DubboService
public class HelloServiceImpl implements HelloService {
    @Override
    public String hello() {
        return "dubbo-provider:hello!";
    }
}
```

#### 2.3.3 服务消费者配置

引入依赖
```xml
<dependency>
    <groupId>org.apache.dubbo</groupId>
    <artifactId>dubbo-spring-boot-starter</artifactId>
    <version>${dubbo-spring-boot.version}</version>
</dependency>
```
application 配置
```yml
spring:
  application:
    # 应用名称
    name: dubbo-consumer-service
  profiles:
    # 环境配置
    active: dev

dubbo:
  application:
    logger: slf4j
    # 关闭qos端口避免单机多生产者端口冲突 如需使用自行开启
    qos-enable: false
    # 元数据中心 local 本地 remote 远程 这里使用远程便于其他服务获取
    metadataType: remote
    # 可选值 interface、instance、all，默认是 all，即接口级地址、应用级地址都注册
    register-mode: instance
    service-discovery:
      # FORCE_INTERFACE，只消费接口级地址，如无地址则报错，单订阅 2.x 地址
      # APPLICATION_FIRST，智能决策接口级/应用级地址，双订阅
      # FORCE_APPLICATION，只消费应用级地址，如无地址则报错，单订阅 3.x 地址
      migration: FORCE_APPLICATION
  protocol:
    # 使用 dubbo 协议通信
    name: dubbo
    # dubbo 协议端口(-1表示自增端口,从20880开始)
    port: -1
  # 注册中心配置
  registry:
    address: nacos://${spring.cloud.nacos.server-addr}
    group: DUBBO_GROUP
    parameters:
      namespace: ${spring.profiles.active}
  # 消费者相关配置
  consumer:
    # 结果缓存(LRU算法)
    # 会有数据不一致问题 建议在注解局部开启
    cache: false
    # 支持校验注解
    # validation: jvalidationNew
    # 调用重试 不包括第一次 0为不需要重试
    retries: 0
    # 初始化检查
    check: false
    # 超时时间
    timeout: 3000
```
定义服务消费者
```java
@RestController
public class HelloController {
    private static final Logger logger = LoggerFactory.getLogger(HelloController.class);
    @DubboReference
    private HelloService helloService;

    @GetMapping("/hello")
    public String hello() {
        logger.info(helloService.hello());
        return "dubbo-consumer:hello!";
    }
}
```

> 服务消费者 No provider available 问题：①排查服务是否在注册中心注册，在服务提供方 application 配置中需指定扫描服务的包路径（dubbo.scan.base-packages）；②提供方及消费者定义的服务（HelloService）名称是否一致（即类名和包名一致，可将其定义为一个公共的接口）。