---
layout:     post
title:      分布式事务-mybatisplus
subtitle:   mybatisplus 的分布式事务支持
date:       2019-09-24
author:     lijian
header-img: img/post-bg-hacker.jpg
catalog: true
tags:
    - 事务
    - mybaits-plus
---

### 简介
mybatis-plus 提供了一个简单的基于mq 的分布式事务消息,但是需要注意的是他其实就是一个正常的队列,只不过是持久化等做了处理.实际上并不是一个标准的分布式事务

官方并不支持强一致性事务：
https://gitee.com/baomidou/dynamic-datasource-spring-boot-starter/wikis/%E5%B8%B8%E8%A7%81%E9%97%AE%E9%A2%98?sort_id=1030627
### 开发

> 暂时支持 rabbit 实现可靠消息分布式事务 3.1.1 以上版本

#### 引入依赖
```java
            <dependency>
                <groupId>com.baomidou</groupId>
                <artifactId>mybatis-plus-dts</artifactId>
                <version>${mybatisplus.version}</version>
            </dependency>
```
#### 自动配置
启动添加 @EnableDtsRabbit 注解

#### 编写消息处理器

实现 IDtsListener 编写消息处理器
```java
@Component
public class RabbitReceiver implements IDtsListener {
    public static final Logger logger = LoggerFactory.getLogger(RabbitReceiver.class);
    @Autowired
    protected PlatformTransactionManager transactionManager;

    @Override
    public void process(DtsMeta dtsMeta) {
        logger.info("Receiving message: {} with transaction manager: {}",
                dtsMeta.getPayload(), transactionManager.getClass().getSimpleName());
        /**
         * 根据 event 处理不同业务逻辑
         */
        if (dtsMeta.getEvent().startsWith("Error")) {
            throw new RuntimeException("Test receiver exception");
        }
    }
}
```
#### 发送消息

事务方法中发送消息

```java
@RestController
@RequestMapping("/rabbit")
public class RabbitController {
    public static final Logger logger = LoggerFactory.getLogger(RabbitController.class);
    @Autowired
    protected PlatformTransactionManager transactionManager;
    @Autowired
    private RabbitRmtSender rmtSender;


    @RequestMapping(value = "/send")
    @Transactional
    public String send() {
        String event = "Message: send";
        logger.info("Sending message: {} with transaction manager: {}", event, transactionManager.getClass().getSimpleName());
        rmtSender.send(new DtsMeta().setEvent(event).setPayload("rabbit send"));
        return String.format("Event sent: %s", event);
    }

    @RequestMapping(value = "/send-error")
    @Transactional
    public String sendError() {
        String event = "Message: sendError";
        logger.info("Sending message: {} with transaction manager: {}", event, transactionManager.getClass().getSimpleName());
        rmtSender.send(new DtsMeta().setEvent(event).setPayload("rabbit send-error"));
        throw new RuntimeException("Test exception");
    }

    @RequestMapping(value = "/send-receive-error")
    @Transactional
    public String sendReceiveError() {
        String event = "ErrorMessage: sendReceiveError";
        logger.info("Sending message: {} with transaction manager: {}", event, transactionManager.getClass().getSimpleName());
        rmtSender.send(new DtsMeta().setEvent(event).setPayload("rabbit send-receive-error"));
        return String.format("Event sent: %s", event);
    }
}
```

### 官网

[官网示例](https://gitee.com/baomidou/mybatis-plus-samples/tree/master/mybatis-plus-sample-dts-rabbit)