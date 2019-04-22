---
layout:     post
title:      Linux 计划任务 crontab
subtitle:   实验楼 Linux 基础学习笔记
date:       2017-09-22
author:     lijian
header-img: img/post-bg-hacker.jpg
catalog: true
tags:
    - Linux
---

[文章 实验楼实验笔记](https://www.shiyanlou.com/courses/1/labs/1918/document/)

### 简介
crontab 储存的指令被守护进程激活，crond 为其守护进程，crond 常常在后台运行，每一分钟会检查一次是否有预定的作业需要执行。这里需要注意的是，每一分钟执行一次
```java
# Example of job definition:
# .---------------- minute (0 - 59)
# |  .------------- hour (0 - 23)
# |  |  .---------- day of month (1 - 31)
# |  |  |  .------- month (1 - 12) OR jan,feb,mar,apr ...
# |  |  |  |  .---- day of week (0 - 6) (Sunday=0 or 7) OR sun,mon,tue,wed,thu,fri,sat
# |  |  |  |  |
# *  *  *  *  * user-name command to be executed
```

### 使用
启动crontab
```java
sudo cron －f &
```
编辑命令
```java
crontab -e
```
添加一个命令
```java
*/1 * * * * touch /home/lijian/$(date +\%Y\%m\%d\%H\%M\%S)

```
显示命令
```java
crontab -l 
```

注意并不是命令添加了就一定会执行，需要有 cron 守护进程在执行才可以，我们可以通过命令查看cron 守护进程是否执行
```java
➜  ~ ps -ef | grep cron
  501  7445  3514   0  3:24下午 ttys004    0:00.00 grep --color=auto --exclude-dir=.bzr --exclude-dir=CVS --exclude-dir=.git --exclude-dir=.hg --exclude-dir=.svn cron
➜  ~
➜  ~ pgrep cron
```

### 深入
/etc/cron.daily，目录下的脚本会每天执行一次，在每天的6点25分时运行；
/etc/cron.hourly，目录下的脚本会每个小时执行一次，在每小时的17分钟时运行；
/etc/cron.monthly，目录下的脚本会每月执行一次，在每月1号的6点52分时运行；
/etc/cron.weekly，目录下的脚本会每周执行一次，在每周第七天的6点47分时运行；