---
layout:     post
title:      ElasticSearch 并发冲突问题理解
subtitle:   ElasticSearch 并发冲突问题理解
date:       2018-04-26
author:     lijian
header-img: img/post-bg-hacker.jpg
catalog: true
tags:
    - ElasticSearch
---


#### 并发冲突问题
 线程1 ，线程2 同时修改一个document （库存-1，原100） ，假设线程1 先修改成功99，线程2 修改为99 就是错误的，es 中应该为 98 
 
#### 悲观锁
* 1.线程1 读取数据 document， 加锁，返回数据
* 2.线程2 读取数据 document, 读取不到
* 3.线程1 减库存，写入数据，线程2 可以读取到最新的 document 
> 悲观锁就是在各种情况下都上锁，在各种条件下上的锁不同，行锁，表锁，读锁，写锁
#### 乐观锁
> es 中用的实际是乐观锁的控制方案
* 线程1 读取document ， 返回数据 (100库存)和 version =1
* 线程2 读取document , 返回数据 （100库存）和 version =1
* 线程1 尝试修改库存为 99,version=2 where version=1 ， 修改成功
* 线程2 尝试修改库存为99 ，version=2 wehre version=1 , 修改失败，因为现在的version =1 
* 线程2 重新读取数据，返回数据 （99库存）和 version =2
* 线程2 尝试修改库存为 98,version=3 where version=2 ， 修改成功
```java
PUT /test_index/test_type/2?version=1  # version = 1 才修改
{
    "test_content":"test"
}
```
 
#### external version 乐观锁控制
* 如果应用程序自己维护了 version , 可以不使用 es 提供的 version 进行乐观锁控制
```java
PUT /test_index/test_type/2?version=1&version_type=external  # version = 1 才修改
{
    "test_content":"test"
}
```