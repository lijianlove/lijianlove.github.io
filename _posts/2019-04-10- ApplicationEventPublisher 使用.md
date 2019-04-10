---
layout:     post
title:      ApplicationEventPublisher 使用
subtitle:   spring 提供的异步框架
date:       2019-04-10
author:     lijian
header-img: img/post-bg-hacker.jpg
catalog: true
tags:
    - 异步
---

# 初识
* 需求：对于拥有某个注解的方法，记录其上下文信息，操作日志希望是异步记录
* 监听器、发布、订阅

# 使用

> 1. 定义 ApplicationEventPublisher 对象，可以直接 @Autowired 等。要发布的事件对象可以是普通 Object 或者 ApplicationEvent
> 2. EventListener 监听事件实现

* ![20190410ApplicationEventListener.png](https://i.loli.net/2019/04/10/5cad9d54ac419.png)
直接使用 ApplicationEventPublisher 的 publishEvent 发送一个 Object 对象 ， 左边的小图标可以直接去到对象的监听实现
* ![20190419ApplicationEvent02.png](https://i.loli.net/2019/04/10/5cad9d9c1fbec.png)

* ![20190419ApplicationEvent03.png](https://i.loli.net/2019/04/10/5cad9e2689d00.png)
通过 EventListener 以及 Async 实现异步监听