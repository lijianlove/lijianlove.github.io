---
layout:     post
title:      SpringFactoriesLoader 初识
subtitle:   SpringFactoriesLoader 初识
date:       2019-08-10
author:     lijian
header-img: img/post-bg-hacker.jpg
catalog: true
tags:
    - spring
    - springboot
---

### 简介




### spring boot starter 原理


springboot starter 一般都是借助 SpringFactoriesLoader 原理，定义 spring.factories , 然后在里面定义诸如

```java
org.springframework.boot.autoconfigure.EnableAutoConfiguration=\
com.baomidou.dynamic.datasource.spring.boot.autoconfigure.DynamicDataSourceAutoConfiguration
```

具体的实现一般定义了一系列的 @Bean 或者 @Import

这样当我们引用starter 的时候就会自动注入相关的配置bean

