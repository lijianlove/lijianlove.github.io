---
layout:     post
title:      ElasticSearch 元数据
subtitle:   ElasticSearch 元数据
date:       2018-04-22
author:     lijian
header-img: img/post-bg-hacker.jpg
catalog: true
tags:
    - ElasticSearch
---


#### 数据结构

```java
{
  "_index" : "ecommerce",
  "_type" : "product",
  "_id" : "1",
  "_version" : 2,
  "_seq_no" : 4,
  "_primary_term" : 1,
  "found" : true,
  "_source" : {
    "name" : "jiaqiangban gaolujie yangao",
    "desc" : "gaoxiao meibai",
    "price" : 30,
    "producer" : "gaolujie producer",
    "tags" : [
      "meibai",
      "fangzhu"
    ]
  }
}

```
* _index 
* 1.document 所在的索引
* 2.

* _type
* 1.document 所在的type
