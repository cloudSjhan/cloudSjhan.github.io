---
title: mysql中字段名与保留字冲突
tags: [MySQL]
copyright: true
date: 2018-12-04 15:24:50
permalink:
categories: MySQL
description: 不小心将MySQL的字段与保留字冲突的解决方法
image: https://static001.geekbang.org/resource/image/f6/f5/f613f6d2d8a72032c16b211f933c1cf5.jpg
---
<p class="description"></p>

<img src="https://" alt="" style="width:100%" />

<!-- more -->

- 在设计数据库的时候不小心将数据库的字段设置成了其内置的保留字，例如下面的这段：

```mysql
CREATE TABLE IF NOT EXISTS `test_billing` (
`vendor` varchar(255),
`cn` varchar(255),
`current_date` varchar(255),
`cost` varchar(255)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;

```

- 这样你在执行类似下面的查询时：

```mysql
select cn, cost from test_billing where current_date like"2018-10%";
```

返回值中什么都没有，还带了一个warning：

```
Empty set, 1 warning (0.00 sec)
```

原因就是字段current_date与MySQL内置的保留字冲突了，那么这时候你还急需查看这些数据，比较快的方法就是：在冲突字段上加反引号 `current_date`,即

```
select cn, cost from test_billing where `current_date` like"2018-10%";
```

就可以解决了。

<hr />