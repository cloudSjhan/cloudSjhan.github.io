---
title: MySQL使用group by分组后对每组操作
tags: [MySQL]
copyright: true
date: 2018-11-05 16:54:14
permalink:
categories: MySQL
description: 在MySQL中使用group by对字段进行分组，并对每组进行统计操作
image:
---
<p class="description"></p>
<!-- more -->

## group by 操作

- 分组能够将数据分成几个逻辑组，然后对其进行聚集操作

- 前几天开发的时候遇到这样的一个问题，有一个vender-cost表：

mysql> select * from vendor-cost;
+---------+--------------+--------------+-----------+------------+----------+----------+

| vendor | host | vendor_id | start_date | cost |
| ------ | ---- | --------- | ---------- | ---- |
|        |      |           |            |      |
+---------+--------------+--------------+-----------+------------+----------+----------+
| Tencent | ins-m9faipc4 | 100014390 | 2018-10 | 0.015456 |
| ------- | ------------ | --------- | ------- | -------- |
|         |              |           |         |          |
| Tencent | ins-r76jxurv | 100015923 | 2018-10 | 0.284697 |
| ------- | ------------ | --------- | ------- | -------- |
|         |              |           |         |          |
| Tencent | ins-ramdkuqz | 100015923 | 2018-10 | 0.021175 |
| ------- | ------------ | --------- | ------- | -------- |
|         |              |           |         |          |
| Tencent | ins-q7o1dhsa | 100014390 | 2018-10 | 0.113501 |
| ------- | ------------ | --------- | ------- | -------- |
|         |              |           |         |          |
| Tencent | ins-5xxrgd65 | 100015923 | 2018-10 | 0.058623 |
| ------- | ------------ | --------- | ------- | -------- |
|         |              |           |         |          |
| Tencent | ins-79g28kn6 | 100015923 | 2018-10 | 0.03808 |
| ------- | ------------ | --------- | ------- | ------- |
|         |              |           |         |         |
| Tencent | ins-rw54ka4k | 100015923 | 2018-10 | 0.150595 |
| ------- | ------------ | --------- | ------- | -------- |
|         |              |           |         |          |
| Tencent | ins-ggxrtm1v | 100015923 | 2018-10 | 0.068281 |
| ------- | ------------ | --------- | ------- | -------- |
|         |              |           |         |          |
为了统计出每个vendor_id的cost，就需要使用分组语句，将同一个vendor_id的cost求和：

select vendor_id, sum(cost) from vendor_cost group by vendor_id;

![](https://ws3.sinaimg.cn/large/006tNbRwly1fwy3y3ulp2j30lk0dejsn.jpg)

得出的结果就是每个vendor_id的总cost。

- 还有一种group by的用法：**GROUP BY X, Y意思是将所有具有相同X字段值和Y字段值的记录放到一个分组里。**
- 举个栗子：

现在有表格

Table: Subject_Selection
Subject   Semester   Attendee
---------------------------------
ITB001    1          John
ITB001    1          Bob
ITB001    1          Mickey
ITB001    2          Jenny
ITB001    2          James
MKB114    1          John
MKB114    1          Erica

- 我们下面再接着要求统计出每门学科每个学期有多少人选择，应用如下SQL

```sql
SELECT Subject, Semester, Count(*)
FROM Subject_Selection
GROUP BY Subject, Semester
```



- 得到的结果是：

### 得到的结果是：

```sql
Subject    Semester   Count
------------------------------
ITB001     1          3
ITB001     2          2
MKB114     1          2
```

- 从表中的记录我们可以看出这个分组结果是正确的有3个学生在第一学期选择了ITB001, 2个学生在第二学期选择了ITB001,还有两个学生在第一学期选择了MKB114, 没人在第二学期选择MKB114。

<hr />
