---
layout:     post
title:      ElasticSearch mget
subtitle:   ElasticSearch mget 批量查询
date:       2018-05-05
author:     lijian
header-img: img/post-bg-hacker.jpg
catalog: true
tags:
    - ElasticSearch
---


#### mget
```java

GET /_mget
{
  "docs":[
    {
      "_index":"test_index",
      "_type":"text_type",
      "_id":1
    },
    {
      "_index":"test_index2",
      "_type":"text_type2",
      "_id":2
    }
    ]
}
GET /test_index/_mget
{
  "docs":[
    {
      "_type":"text_type",
      "_id":1
    }
    ]
}
GET /test_index/_type/_mget
{
  "ids":[1,2,10]
}
```