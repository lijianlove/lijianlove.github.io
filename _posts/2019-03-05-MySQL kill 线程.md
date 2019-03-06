---
layout:     post
title:      MySQL kill 线程
subtitle:   MySQL 解决死锁
date:       2019-03-05
author:     lijian
header-img: img/post-bg-hacker.jpg
catalog: true
tags:
    - MySQL
    - 数据库
---


### 获取连接列表


* show processlist;

[![]({{site.url}}/img/201903/sql20190305.png)]()

### 杀掉线程
* kill 43016408
