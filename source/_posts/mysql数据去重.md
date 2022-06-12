---
title: mysql数据去重
tags: [MySQL]
copyright: true
date: 2019-07-01 15:40:05
permalink:
categories: MySQL
description: mysql数据去重
image: https://static001.geekbang.org/resource/image/cd/ba/cd90547adb10141e3a6eb969d9c0b5ba.jpg
---
<p class="description"></p>
<img src="https://" alt="" style="width:100%" />

<!-- more -->

从excel中导入了一部分数据到mysql中，有很多数据是重复的，而且没有主键，需要按照其中已经存在某一列对数据进行去重。

## 添加主键

由于之前的字段中没有主键，所以需要新增一个字段，并且将其作为主键。

添加一个新的字段id，对id中的值进行递增操作，然后再设置为主键。

对id字段进行递增的赋值操作如下：

```
SET @r:=0;
UPDATE table SET id=(@r:=@r+1);
```

然后设置为主键即可。

## 去重

添加递增的id字段后，就可以对数据根据某个字段进行去重操作，策略就是保存id最小的那条数据。

```mysql
DELETE FROM `table`
WHERE
`去重字段名` IN (
    SELECT x FROM
    (
        SELECT `去重字段名` AS x 
        FROM `table` 
        GROUP BY `去重字段名` 
        HAVING COUNT(`去重字段名`) > 1
    ) tmp0
)
AND 
`递增主键名` NOT IN (
    SELECT y FROM
    (
        SELECT min(`递增主键名`) AS y 
        FROM `table` 
        GROUP BY `去重字段名` 
        HAVING COUNT(`去重字段名`) > 1
    ) tmp1
)
```

<hr />
