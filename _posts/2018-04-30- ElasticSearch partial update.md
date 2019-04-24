---
layout:     post
title:      ElasticSearch partial update
subtitle:   ElasticSearch partial update
date:       2018-04-30
author:     lijian
header-img: img/post-bg-hacker.jpg
catalog: true
tags:
    - ElasticSearch
---


#### partial update
正常的 update 需要传入所有的字段，然后执行全量替换，标记 deleted , 重建 document 等
partial update 只需要传入关注的字段即可

#### 原理
* 实际和 全量更新原理一样，只不过查询全量document 在 es  自动执行中

内部先获取document , 将 字段放入 查询出的 document ， 然后执行全量替换，标记 deleted , 重建 document 等

#### 基于 groovy 脚本执行 partial update
es 实际是内置脚本支持的，可以基于 groovy 实现复杂的操作

##### 内部脚本
```java
PUT /test_index/test_type/10
{
  "num":0,
  "tags":[]
}
```
```java

POST /test_index/test_type/10/_update
{
  "script": "ctx._source.num+=1"
}
```

```java
{
  "_index" : "test_index",
  "_type" : "test_type",
  "_id" : "10",
  "_version" : 2,
  "_seq_no" : 1,
  "_primary_term" : 1,
  "found" : true,
  "_source" : {
    "num" : 1,
    "tags" : [ ]
  }
}

```
##### 外部脚本
* 脚本如果比较复杂，可以将脚本封装在 es 安装目录/config/scripts/ 下，比如我们新建一个 test-add-tags.groovy
```javascript
ctx._source.tag+=new_tag
```
```javascript
POST /test_index/test_type/10/_update
{
  "script":{
    "lang": "groovy", 
    "file": "test-add-tags",
    "params": {
      "new_tag":"tag1"
    }
  }
}
```
