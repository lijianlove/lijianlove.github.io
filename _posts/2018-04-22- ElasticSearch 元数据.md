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
* 2.类似的数据放在一个index 中,不能将所有的数据放在一个index 中
* 3.索引名称必须是小写的，不能用下划线开头，不能包含都好

* _type
* 1.document 所在的type
* 2.一个索引被划分为多个type,逻辑上对index 中的数据进行分类
* 3.type 名称可以是大写或者小写，不能用下划线开头，不能包含逗号

* _id
* 1.document 的唯一标示，与index 和 type 一起，可以唯一标识和定位一个document
* 2.可以手动指定也可以有es 自动生成

* _source
* 1.在创建 document 时候的 body 串
* 2.可以指定返回字段，如果是多个字段，用逗号分隔
```java
GET /test_index/test_type/2?_source=test_content
```
* _version
* 新创建的时候是 1 ，以后每次修改或者删除都会修改 +1，
* 先删除一个 document , 在创建该 document， version 为 deleted +1 
