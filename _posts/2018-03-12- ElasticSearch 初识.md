---
layout:     post
title:      ElasticSearch 初识
subtitle:   ElasticSearch 初识
date:       2018-03-12
author:     lijian
header-img: img/post-bg-hacker.jpg
catalog: true
tags:
    - ElasticSearch
---


### 引入
* 基于数据库的搜索效率差
* 基于数据库的搜索不能拆分关键字搜索

### ElasticSearch
* 自动维护数据的冗余副本，保证有机器宕机，不丢失数据
* 封装了更多高级功能，提供更高级的查询
* 自动维护数据到多节点的索引的建立，以及检索请求到多节点的执行
* 分布式搜索以及数据分析引擎
* 对海量数据的近实时处理
* 全文检索，结构化检索，数据分析
* 分布式文档数据存储系统
* 适用于 数据量较大（分布式）
* 适用于 数据结构灵活多变，而且数据关系复杂（json 存储）
* 适用于 对数据的相关操作较为简单，比如简单的增删改查

### 与 Lucene 的关系
* 基于 Lucene 开发比较复杂
* ES 是基于 Lucene 开发的，简化开发，暴露简单的开发接口，分布式存储引擎，分布式数据分析引擎，分布式海量级别数据，开箱即用

### 核心概念
* Near Realtime (NRT): 近实时，插入数据到可以搜索到，数据延迟秒级（大约1秒）。基于数据的搜索、分析也是秒级
* Cluster : 集群 ， 包含多个节点，每个集群都有一个名称，默认 elasticsearch
* Node : 节点， 归属于一个集群，节点也有名称，默认是随机分配的
* Index : 索引， 一个 index 包含很多 document ， 一个  index 包含了一类或者相同的document ，每个index 默认配置是5个primary shard , 5 个 replica shard 
* type : 一个index 可以放很多 type , type 就是代表 index 中的一个逻辑上的数据分类，比如一个商品 index， 存放了所有的商品数据document，但是商品分为很多种类，比如电器商品，包含售后时间这样的特殊field，生鲜类食品包含特殊的保质期等field。type 就是用来区分这种分类的，比如电器类 type， 生鲜类type， 每个不同的type包含不同的field
每个type 中包含很多 document 
* Document : 文档，ES 中最小的一个数据单元，比如是一条商品数据，一个订单数据，通常用 json 格式，一个 index 下的type 中都可以包含多个document， 一个document 包含多个field， 每个field 都是一个数据字段
* shard : 单台机器无法存储大量数据，es 可以将一个索引中的数据切分为多个 shard , 每个 shard 存放 index 的一部分数据，分布在多台服务器上部署，有了 shard 就可以横向拓展，存储更多的数据，让搜索和分析等操作分布到多台服务器上去操作，提高吞吐量和性能。每个shard 都是一个 Lucene index
* replica : 任何一台服务器都可能宕机，shard 数据就会丢失，可以为每个 shard 建立多个 replica 副本 （默认一个，可以修改）。保证在shard 宕机时 replica 可以继续堆外提供服务。所以最小的高可用配置是2台服务器。如果一个 shard 宕机，机会造成部分数据丢失，replica 就是一个对应 shard 的副本，保证一个shard 宕机，还可以在另一个 replica 上搜索到该 shard 的数据。提供了高可用，一个 shard 宕机，服务依旧可用。提高了搜索这类请求的吞吐量，也就是说 replica 可以提高对应 shard 的搜索吞吐量。

### mac 操作
* 安装 ： brew install elasticsearch
* 查看基础配置： brew info elasticsearch 
* 查看目录： brew list elasticsearch
* 进入启动目录： cd xxxxx/bin
* 启动es : ./elasticsearch
* 查看是否启动成功: 浏览器打开： http://localhost:9200/?pretty
```js
{
    name: "PASaJyf", # node 名称
    cluster_name: "elasticsearch_bjhl", # 集群名称
    cluster_uuid: "r8ecyo1tTZOZfJez0LA-qA",
    version: {
        number: "6.7.0",
        build_flavor: "oss",
        build_type: "tar",
        build_hash: "8453f77",
        build_date: "2019-03-21T15:32:29.844721Z",
        build_snapshot: false,
        lucene_version: "7.7.0",
        minimum_wire_compatibility_version: "5.6.0",
        minimum_index_compatibility_version: "5.0.0"
    },
    tagline: "You Know, for Search"
}
```

* 安装 kibana： brew install kibana
* 启动 kibana: brew services start kibana
* 查看是 kibana 否启动成功：http://localhost:5601/app/kibana#/home?_g=()
* 在 kibana dev Tools 中验证 ES : GET _cluster/health
```js
{
  "cluster_name" : "elasticsearch_bjhl",
  "status" : "green", # red , yellow , green
  "timed_out" : false,
  "number_of_nodes" : 1,
  "number_of_data_nodes" : 1,
  "active_primary_shards" : 1,
  "active_shards" : 1,
  "relocating_shards" : 0,
  "initializing_shards" : 0,
  "unassigned_shards" : 0,
  "delayed_unassigned_shards" : 0,
  "number_of_pending_tasks" : 0,
  "number_of_in_flight_fetch" : 0,
  "task_max_waiting_in_queue_millis" : 0,
  "active_shards_percent_as_number" : 100.0
}

```

### 简单操作
ES 提供了一套 cat api , 提供简单的集群操作

* 快速查看集群状态
```java
GET /_cat/health?v
```
```js
epoch      timestamp cluster            status node.total node.data shards pri relo init unassign pending_tasks max_task_wait_time active_shards_percent
1555121915 02:18:35  elasticsearch_bjhl green           1         1      1   1    0    0        0             0                  -                100.0%

```
* green : 每个索引的 primary shard 和 relica shard 都是 active 状态的
* yellow : 每个索引的 primary shard 都是 active 状态的，但是部分 replica shard 不是 active 状态，处于不可用状态
* red ： 不是所有 primary shard 都是 active 状态的，部分索引有数据丢失了

* 快速查看集群中所有索引
```java
GET /_cat/indices?v
```
```js
health status index     uuid                   pri rep docs.count docs.deleted store.size pri.store.size
green  open   .kibana_1 hU2uGTDlTP2azlp7gVDXWA   1   0          1            0      3.6kb          3.6kb

```

* 快速创建索引
```java
PUT /test_index?pretty
```
```js
{
  "acknowledged" : true,
  "shards_acknowledged" : true,
  "index" : "test_index"
}
```

```java
DELETE /test_index?pretty
```
```js
{
  "acknowledged" : true
}

```


### 再理解
Java 中面向对象开发，存储一个这样的数据结构
```java
class Employee {
 private String employeeNo;
 private Long salary;
 private Persion info; 
}
```
我们在数据库中存储会完全打平数据，存储两张表，用外键关联
ES 面向document ，用json 存储数据
直接
```js
{
    "employeeNo":"12122",
    "salary":2222222222,
    "info":{
        "name":"zhangsan",
        "sex":"m"
    }
}

```