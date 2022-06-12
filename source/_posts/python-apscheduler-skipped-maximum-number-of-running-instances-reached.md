---
title: 'python apscheduler - skipped: maximum number of running instances reached'
tags: [python]
copyright: true
date: 2018-09-28 16:04:43
permalink:
categories: python
description: 'python apscheduler - skipped: maximum number of running instances reached'
image:
---
<p class="description"></p>

<img src="https://" alt="" style="width:100%" />

<!-- more -->

### 出现问题的代码

```Python
scheduler = BackgroundScheduler()
scheduler.add_job(runsync, 'interval', seconds=1)
scheduler.start()
```

### 问题出现的情况

* 运行一段代码，时而报错时而不报错
* 报错是：

```
WARNING:apscheduler.scheduler:Execution of job "runsync (trigger: interval[0:00:01], next run at: 2015-12-01 11:50:42 UTC)" skipped: maximum number of running instances reached (1)
```

### 分析

* apscheduler这个模块，在你的代码运行时间大于interval的时候，就会报错

  也就是说，你的代码运行时间超出了你的定时任务的时间间隔。

### 解决

* 增大时间间隔即可

### 

<hr />
