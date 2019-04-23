---
layout:     post
title:      ElasticSearch ID 自动生成和手动指定
subtitle:   ElasticSearch ID 自动生成和手动指定
date:       2018-04-22
author:     lijian
header-img: img/post-bg-hacker.jpg
catalog: true
tags:
    - ElasticSearch
---


#### 手动指定
* 从其他系统中导入到es 会采取手动指定的方式，采用系统已有的唯一标示作为es document的 _id 

```java
PUT /test_index/test_type/2
{
  "test_content":"this is content"
}
```
```java
GET /test_index/test_type/2
```
```java
{
  "_index" : "test_index",
  "_type" : "test_type",
  "_id" : "2",  # 这里的_id 是我们指定的
  "_version" : 2,
  "_seq_no" : 1,
  "_primary_term" : 1,
  "found" : true,
  "_source" : {
    "test_content" : "this is content"
  }
}

```
#### 自动生成
* es 就是数据的唯一存储介质
* 自动生成ID程度为20个字符
* url 安全的，可以放在url 中
* guid , 分布式系统并行生成也不会发生冲突

```java
POST /test_index/test_type
{
  "test_content":"this is NEW content"
}

```
```java
{
  "_index" : "test_index",
  "_type" : "test_type",
  "_id" : "jREORWoBnWNfF8XNxH71", # 注意这里，_id 为我们自动生成的ID
  "_version" : 1,
  "result" : "created",
  "_shards" : {
    "total" : 2,
    "successful" : 1,
    "failed" : 0
  },
  "_seq_no" : 0,
  "_primary_term" : 1
}

```