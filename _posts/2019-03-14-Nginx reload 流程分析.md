---
layout:     post
title:      Nginx reload 流程分析
subtitle:   Nginx -s reload 流程分析
date:       2019-03-14
author:     lijian
header-img: img/post-bg-hacker.jpg
catalog: true
tags:
    - Nginx
---
#### 简单流程

* worker 告诉 master ，我不干啦，你找新人吧
* master 说，好的，从现在开始，你就是老人啦
* master 用新的 config 去创建新的 worker 进程
* master 告诉老 worker , 从现在起，你就不要接新任务啦
* 老 worker 忙完自己的活
* 老 worker 关闭

#### 详细流程


[![]({{site.url}}/img/201903/20190314nginx_reload.png)]()