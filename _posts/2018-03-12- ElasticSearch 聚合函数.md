---
layout:     post
title:      ElasticSearch 聚合函数
subtitle:   ElasticSearch 聚合函数
date:       2018-03-12
author:     lijian
header-img: img/post-bg-hacker.jpg
catalog: true
tags:
    - ElasticSearch
---

* 使用aggs 计算每个tag 下的商品数量
```java

GET /ecommerce/product/_search
{
  "aggs": {
    "myname_groupbytags": {
      "terms": {
        "field": "tags"
      }
    }
  }
}

```
```js
{
  "error": {
    "root_cause": [
      {
        "type": "illegal_argument_exception",
        "reason": "Fielddata is disabled on text fields by default. Set fielddata=true on [tags] in order to load fielddata in memory by uninverting the inverted index. Note that this can however use significant memory. Alternatively use a keyword field instead."
      }
    ],
    "type": "search_phase_execution_exception",
    "reason": "all shards failed",
    "phase": "query",
    "grouped": true,
    "failed_shards": [
      {
        "shard": 0,
        "index": "ecommerce",
        "node": "PASaJyfGTv-eGqVgFgNMyg",
        "reason": {
          "type": "illegal_argument_exception",
          "reason": "Fielddata is disabled on text fields by default. Set fielddata=true on [tags] in order to load fielddata in memory by uninverting the inverted index. Note that this can however use significant memory. Alternatively use a keyword field instead."
        }
      }
    ],
    "caused_by": {
      "type": "illegal_argument_exception",
      "reason": "Fielddata is disabled on text fields by default. Set fielddata=true on [tags] in order to load fielddata in memory by uninverting the inverted index. Note that this can however use significant memory. Alternatively use a keyword field instead.",
      "caused_by": {
        "type": "illegal_argument_exception",
        "reason": "Fielddata is disabled on text fields by default. Set fielddata=true on [tags] in order to load fielddata in memory by uninverting the inverted index. Note that this can however use significant memory. Alternatively use a keyword field instead."
      }
    }
  },
  "status": 400
}
```
将 tags 字段的 fielddata 设置为true
```java
PUT /ecommerce/_mapping/product
{
  "properties": {
    "tags":{
      "type": "text",
      "fielddata": true
    }
  }
}
```
```js
{
  "acknowledged" : true
}
```
再次执行聚合函数
```js
{
  "took" : 341,
  "timed_out" : false,
  "_shards" : {
    "total" : 5,
    "successful" : 5,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : 3,
    "max_score" : 1.0,
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
          "producer" : "jiaojieshi producer",
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
          "producer" : "zhonghua producer",
          "tags" : [
            "meibai",
            "fangzhu"
          ]
        }
      }
    ]
  },
  "aggregations" : {
    "myname_groupbytags" : {
      "doc_count_error_upper_bound" : 0,
      "sum_other_doc_count" : 0,
      "buckets" : [
        {
          "key" : "fangzhu",
          "doc_count" : 3
        },
        {
          "key" : "meibai",
          "doc_count" : 3
        }
      ]
    }
  }
}

```

* 对名称中包含yagao 的商品计算每个tag 的数量
```js
GET /ecommerce/product/_search
{
  "size": 0, # size 0 可以去除掉每个数据项的输出，只关心统计结果
  "query": {
    "match": {
      "name": "yagao"
    }
  }, 
  "aggs": {
    "myname_groupbytags": {
      "terms": {
        "field": "tags"
      }
    }
  }
}
```

* 先分组，在算每组的平均值，计算每个tag 下商品的平均价格
```java
GET /ecommerce/product/_search
{
  "size": 0, 
  "aggs": {
    "myname_groupbytags": {
      "terms": {
        "field": "tags"
      },
      "aggs":{
        "myname_avgprice":{
          "avg": {
            "field": "price"
          }
        }
      }
    }
  }
}
```

```js
{
  "took" : 10,
  "timed_out" : false,
  "_shards" : {
    "total" : 5,
    "successful" : 5,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : 3,
    "max_score" : 0.0,
    "hits" : [ ]
  },
  "aggregations" : {
    "myname_groupbytags" : {
      "doc_count_error_upper_bound" : 0,
      "sum_other_doc_count" : 0,
      "buckets" : [
        {
          "key" : "meibai",
          "doc_count" : 3,
          "myname_avgprice" : {
            "value" : 33.333333333333336
          }
        },
        {
          "key" : "fangzhu",
          "doc_count" : 2,
          "myname_avgprice" : {
            "value" : 35.0
          }
        },
        {
          "key" : "fangzhula",
          "doc_count" : 1,
          "myname_avgprice" : {
            "value" : 30.0
          }
        }
      ]
    }
  }
}

```

* 按照 tags 进行分组，计算平均价格，按照降序排序
```java
GET /ecommerce/product/_search
{
  "size": 0, 
  "aggs": {
    "myname_groupbytags": {
      "terms": {
        "field": "tags",
        "order": {
          "myname_avgprice": "desc"
        }
      },
      "aggs":{
        "myname_avgprice":{
          "avg": {
            "field": "price"
          }
        }
      }
    }
  }
}
```
* 按照指定的价格范围进行分组，在每组内再按照tag 进行分组，再计算每组的平均价格
```java
GET /ecommerce/product/_search
{
  "size": 0, 
  
  "aggs": {
    "myname_groupprice":{
      "range": {
        "field": "price",
        "ranges": [
          {
            "from": 0,
            "to": 20
          },
          {
            "from": 20,
            "to": 30
          },
          {
            "from": 30,
            "to": 40
          }
        ]
      },
      "aggs": {
        "myname_groupbytags": {
            "terms": {
              "field": "tags",
              "order": {
                "myname_avgprice": "desc"
              }
            },
            "aggs":{
              "myname_avgprice":{
                "avg": {
                  "field": "price"
                }
              }
            }
        }
      }
    }
    
  }
}
```