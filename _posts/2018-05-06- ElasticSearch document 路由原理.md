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

#### document 增删改原理
一个操作 document 请求打入到任意一个node  上，该node 就是 coordinate node (协调节点)， 该node 基于上面的路由算法进行路由，可以知道应该哪个 primary shard , 再将该请求发送到对应的primary shard 上, primary shard 处理完毕 document 之后，就会将数据同步到自己的 replica shard 上去，coordinate node 发现 primary shard , replica shard 都处理完毕，就会将结果反馈给客户端

对于增删改操作，只能由 primary shard 进行处理，不能由 replica shard 进行处理。

#### document 查询原理
对于读请求，不一定由 primary shard 进行处理，可能有 replica shard 进行处理，采用 round-robin 随机轮询算法，尽量让 primary shard 和 replica shard 均匀的负载请求

#### 写一致性
我们在发送任何一个增删改操作的时候，我们可以带上一个参数 consistency ，指明想要的一致性。 比如 one, all , quorum
```java
PUT /index/type/id?consistency=one
```

* one 只要有一个 primary shard 是 active 活跃可用的就可以执行。
* all 必须所有的 primay shard 和 replica shard 都是活跃的才可以执行
* quorum 默认就是它，要所有的shard 中必须大部分是活跃可用的才可以执行，但是只有当 number_of_replica 大于1 才会生效

quonum=((primary + number_of_replicas)/2)+1
* quonum 不齐全，默认 wait 1分钟，最后 timeout,我们可以在写操作的时候加一个timeout 参数,单位默认是毫秒，比如 30毫秒
```java
PUT /index/type/id?timeout=30
```
* 如果节点数小于 quonum , 可能导致 quonum 不全，导致无法进行任何写操作



