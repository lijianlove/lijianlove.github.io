---
layout:     post
title:      ElasticSearch 全量替换、强制创建以及lazy delete 机制 
subtitle:   ElasticSearch 全量替换、强制创建以及lazy delete 机制 
date:       2018-04-25
author:     lijian
header-img: img/post-bg-hacker.jpg
catalog: true
tags:
    - ElasticSearch
---


#### 全量替换
* 全量替换与创建的语法是一样的，如果document _id 不存在，就创建，如果存在就修改
* document 是不可变的，要想修改，就是要全量替换，直接对document 重建索引，替换所有内容
* es 会将老的document 标记为 deleted，然后给我们一个新的 document , 当创建越来越多document 的时候，es 会在合适的时机删除 deleted document

#### 强制创建
* 强制创建与全量替换的语法是一样的
* PUT /index/type/id/_create   PUT /index/type/id?op_type=create


#### 删除document
* DELETE /index/type/id
* 只是标记为deleted , 并没有物理删除，在合适的时机才会删除
