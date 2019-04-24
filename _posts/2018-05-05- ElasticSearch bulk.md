---
layout:     post
title:      ElasticSearch bulk
subtitle:   ElasticSearch bulk 批量增删改
date:       2018-05-05
author:     lijian
header-img: img/post-bg-hacker.jpg
catalog: true
tags:
    - ElasticSearch
---


#### bulk
* 注意： bulk api 对 json 有严格的要求，每个json 不能换行，只能放一行，同时，两个json 之间必须有换行
* bulk 操作中， 任意一个操作失败是不会影响其他操作的，但是在返回结果中会告诉对应的异常日志
* bulk request 会加载到内存里，如果request 太大的话反而会影响性能
```java
POST /my_index/my_type/_bulk
{ "index": { "_id": 1 }}
{ "title": "The quick brown fox" }
{ "index": { "_id": 2 }}
{ "title": "The quick brown fox jumps over the lazy dog" }
{ "index": { "_id": 3 }}
{ "title": "The quick brown fox jumps over the quick dog" }
{ "index": { "_id": 4 }}
{ "title": "Brown fox brown dog" }
```
```java
POST /_bulk
{ "delete":{ "_index":"my_index", "_type":"my_type",      "_id":"2"}}
{ "create":{ "_index":"my_index", "_type":"my_type",   "_id":"15" }}
{ "title":"this is bulk create"}
{ "update":{"_index":"my_index","_type":"my_type","_id":"3"}}
{ "doc":{"title":"this is replaced title"}}
{ "index":{"_index":"my_index","_type":"my_type","_id":"2"}}
{ "title":"this is replaced title"}

```