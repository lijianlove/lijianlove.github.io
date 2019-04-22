---
layout:     post
title:      ElasticSearch 分布式架构
subtitle:   ElasticSearch 分布式架构
date:       2018-03-15
author:     lijian
header-img: img/post-bg-hacker.jpg
catalog: true
tags:
    - ElasticSearch
---


* 增加或者删除节点，数据 reblance 保持负载均衡

* master 节点
* 1. 创建和删除索引
* 2. 创建和删除节点

* 节点平等的分布式概念
* 1.节点对等，每个节点都能接受请求
* 2.自动请求路由

### primary shard 和 replica shard 再梳理

* 一个index 包含多个 shard 
* 每个shard 都是一个最小工作单元，都是一个 Lucene 实例，承载部分数据，有完整的建立和处理请求的能力
* 增加节点时，shard 自动在nodes 中负载均衡
* 每个document 存在一个primary shard 及其对应的 replica shard ， 一个document 不可能存在于多个不同的 primary shard
* replica shard 是 primary shard 的数据副本以及承载部分数据请求
* replica shard 不能和自己对应的 primary shard 在一台机器上

