---
layout:     post
title:      ElasticSearch document 路由原理
subtitle:   ElasticSearch document 路由原理
date:       2018-05-06
author:     lijian
header-img: img/post-bg-hacker.jpg
catalog: true
tags:
    - ElasticSearch
---


#### 来源

一个index 的数据会被分为多片，每个分片存在于一个shard 中，对于一个document 应该放在 哪个 primary shard 上呢？ 

#### 解决

> shard=hash（routing）%number_of_primary_shards

比如一个 index 有三个 primary shard ， 
当我们增删改查一个document 的时候，默认都会带过来一个 routing number ， 默认是 document 的 _id (可能是手动指定，也可能是自动生成)
再根据 routing number 以及 shard=hash（routing）%number_of_primary_shards 算出一个 primary shard , 将该 document 的相关操作打入对应的 shard 上。
就是说决定一个document 在哪个shard 上最重要的就是 document 的_id , 无论hash 值 是什么，对 number_of_primary_shards 取余，结果一定在 0-number_of_primary_shards-1 之间

也可以在发送请求的时候，手动指定一个 routing number , 手动指定 routing number 是非常有用的，可以保证某一类数据在一个 primary shard 上存储

> PUT /index/type/1?routing=user_id

#### number_of_primary_shards 不可变

因为路由公式基于这个数，如果这里改变，那么就会导致 shard 计算有问题，会导致document 路由到错误的primary shard , 所以这里不可变，replica shard 是可以修改的。学习到这里，想到了 redis 的的处理机制，数据基于槽，机器数量改变会移动槽中的数据

#### document 增删改查原理
