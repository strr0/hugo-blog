---
title: "Spring 事件驱动"
date: 2023-11-08T10:00:00+08:00
categories: ["development"]
tags: ["java"]
draft: false
---

## 介绍

事件驱动是指在持续事务管理过程中，进行决策的一种策略，即跟随当前时间点上出现的事件，调动可用资源，执行相关任务，使不断出现的问题得以解决，防止事务堆积。

## 1 示例

### 1.1 定义事件

事件类继承 ApplicationEvent（非必须）

```
public class MyEvent extends ApplicationEvent {
    public MyEvent(Object source) {
        super(source);
    }
}
```

### 1.2 定义事件监听器

#### 1.2.1 通过实现接口方式

```
public class MyListener implements ApplicationListener<MyEvent> {
    @Override
    public void onApplicationEvent(MyEvent event) {
        // TODO
    }
}
```

#### 1.2.2 通过注解方式

```
@Component
public class MyListener {
    @EventListener
    public void onReceiveEvent(MyEvent event) {
        // TODO
    }
}
```

### 1.3 测试

```
@SpringBootTest
public class ApplicationTests {
    @Autowired
    private ApplicationContext applicationContext;

    @Autowired
    private ApplicationEventPublisher applicationEventPublisher;

    @Test
    void test1() {
        applicationContext.publishEvent(new MyEvent("my-event"));
    }

    @Test
    void test2() {
        applicationEventPublisher.publishEvent(new MyEvent("my-event"));
    }
}
```

## 2 其他问题

### 2.1 异步执行

事件驱动默认是同步的，事件发布者会阻塞等待事件处理完成，可通过 Async 注解设置为异步执行
```
@Component
public class MyListener {
    @Async
    @EventListener
    public void onReceiveEvent(MyEvent event) {
        // TODO
    }
}
```