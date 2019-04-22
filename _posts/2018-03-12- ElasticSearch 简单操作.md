---
layout:     post
title:      ElasticSearch 简单操作
subtitle:   ElasticSearch 简单操作
date:       2018-03-12
author:     lijian
header-img: img/post-bg-hacker.jpg
catalog: true
tags:
    - ElasticSearch
---

### CRUD

* 新增和修改都可以用这个，但是修改如果用这个命令必须带所有field PUT /index/type/id
```js
PUT /ecommerce/product/1
{
    "name":"gaolujie yagao",
    "desc":"gaoxiao meibai",
    "price":30,
    "producer":"gaolujie producer",
    "tags":["meibai","fangzhu"]
}
```
```js
{
  "_index" : "ecommerce",
  "_type" : "product",
  "_id" : "1",
  "_version" : 1,
  "result" : "created",
  "_shards" : {
    "total" : 2, # 想要插入 primary shard and replica shard
    "successful" : 1, # 目前急群众只有一个 primary shard ，所以成功是1
    "failed" : 0
  },
  "_seq_no" : 1,
  "_primary_term" : 1
}

```
* GET /index/type/id
```java
GET /ecommerce/product/1
```
```js
{
  "_index" : "ecommerce",
  "_type" : "product",
  "_id" : "1",
  "_version" : 2,
  "_seq_no" : 1,
  "_primary_term" : 1,
  "found" : true,
  "_source" : {
    "name" : "gaolujie yagao",
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
* DELETE /index/type/id
```java
DELETE /ecommerce/product/1
```
```js
{
  "_index" : "ecommerce",
  "_type" : "product",
  "_id" : "1",
  "_version" : 3,
  "result" : "deleted",
  "_shards" : {
    "total" : 2,
    "successful" : 1,
    "failed" : 0
  },
  "_seq_no" : 2,
  "_primary_term" : 1
}

```

* 修改指定字段： POST /index/type/id/_update
```java
POST /ecommerce/product/1/_update
{

  "doc":{
      "name":"jiaqiangban gaolujie yangao"
  }
}
```

```js
{
  "_index" : "ecommerce",
  "_type" : "product",
  "_id" : "1",
  "_version" : 2,
  "result" : "updated",
  "_shards" : {
    "total" : 2,
    "successful" : 1,
    "failed" : 0
  },
  "_seq_no" : 4,
  "_primary_term" : 1
}

```

### 多种搜索方式
#### query string search
* 搜索全部document

GET /index/type/_search

```java
GET /ecommerce/product/_search
```
```js
{
  "took" : 202, # 搜索用时，毫秒
  "timed_out" : false,
  "_shards" : { # 默认5个 shard,所以查询5 
    "total" : 5,
    "successful" : 5,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : 3,  # 匹配结果数
    "max_score" : 1.0, # document 对于 相关度的匹配分数，越相关越匹配分数越高，最大是1
    "hits" : [
      {
        "_index" : "ecommerce",
        "_type" : "product",
        "_id" : "2",
        "_score" : 1.0,
        "_source" : {
          "name" : "jiaojieshi yagao",
          "desc" : "gaoxiao meibai",
          "price" : 30,
          "producer" : "gaolujie producer",
          "tags" : [
            "meibai",
            "fangzhu"
          ]
        }
      },
      {
        "_index" : "ecommerce",
        "_type" : "product",
        "_id" : "1",
        "_score" : 1.0,
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
      },
      {
        "_index" : "ecommerce",
        "_type" : "product",
        "_id" : "3",
        "_score" : 1.0,
        "_source" : {
          "name" : "zhonghua yagao",
          "desc" : "gaoxiao meibai",
          "price" : 40,
          "producer" : "gaolujie producer",
          "tags" : [
            "meibai",
            "fangzhu"
          ]
        }
      }
    ]
  }
}

```
* 查询 name like 'yagao' 按照价格降序排序
```java
GET /ecommerce/product/_search?q=name:yagao&sort=price:desc
```

#### query DSL
DSL: Domain Specified Language (特定领域语言)
http requet body 请求体可以使用 json 格式构建查询语法，可以构建比较复杂的语法，比 query search strign 强大
* 查询所有document : 
```java
GET /ecommerce/product/_search
{
  "query":{
    "match_all":{}
  }
}
```

* 查询 name like 'yagao' 按照价格降序排序
```java
GET /ecommerce/product/_search
{
  "query":{
    "match":{
      "name":"yagao"
    }
  },
  "sort":[
    {    "price":"desc"  }
  ]
}

```
* 分页查询
```java
GET /ecommerce/product/_search
{
  "query":{
    "match_all":{}
  },
  "from":0,
  "size":1
}
```
* 查询指定字段
```java
GET /ecommerce/product/_search
{
  "query":{
    "match_all":{}
  },
  "_source": ["name","desc"], 
  "from":0,
  "size":1
}

```
#### query filter 复杂过滤
```java
GET /ecommerce/product/_search
{
  "query":{
    "bool":{
      "must": {
          "match":{
            "name":"yagao"
          }
      },
      "filter":{
        "range":{
          "price":{
            "gt":25
          }
        }
      }
    }
  },
  "_source": ["name","desc"], 
  "from":0,
  "size":1
}
#### 全文检索
将搜索字符串拆解去倒排索引中去检索，只要可以匹配任意一个单词，就返回
```
* 按照 producer 字段全文检索
```java
GET /ecommerce/product/_search
{
  "query":{
    "match":{
      "producer": "yagao producer"
    }
  }
}
```

#### 短语检索（phrase search）
要求输入的搜索字符串必须在指定的字段中完全匹配才可以返回
```java
GET /ecommerce/product/_search
{
  "query":{
    "match_phrase":{
      "producer": "yagao producer"
    }
  }
}
```
#### hignlignt search 
```java
GET /ecommerce/product/_search
{
  "query":{
    "match_phrase":{
      "producer": "jiaojieshi"
    }
  },
  "highlight": {
    "fields": {
      "producer": {}
    }
  }
}
```
```js
{
  "took" : 434,
  "timed_out" : false,
  "_shards" : {
    "total" : 5,
    "successful" : 5,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : 1,
    "max_score" : 0.2876821,
    "hits" : [
      {
        "_index" : "ecommerce",
        "_type" : "product",
        "_id" : "2",
        "_score" : 0.2876821,
        "_source" : {
          "name" : "jiaojieshi yagao",
          "desc" : "gaoxiao meibai",
          "price" : 30,
          "producer" : "jiaojieshi producer",
          "tags" : [
            "meibai",
            "fangzhu"
          ]
        },
        "highlight" : {  # 注意此处
          "producer" : [
            "<em>jiaojieshi</em> producer"
          ]
        }
      }
    ]
  }
}

```