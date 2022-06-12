---
title: mysql add primary key auto_increment
tags: [MySQL]
copyright: true
date: 2018-12-25 19:08:53
permalink:
categories: MySQL
description: MySQL add primary key auto_increment for established table
image: https://static001.geekbang.org/resource/image/12/5e/12b11ffae84ecbc0f8e8e15586136a5e.jpg
---
<p class="description"></p>

<img src="https://" alt="" style="width:100%" />

<!-- more -->

添加字段id,并将其设置为主键自增

- alter table TABLE_NAME add id int not null primary key Auto_increment

如果想添加已经有了一列为主键，可以用：

- alter table TABLE_NAME add primary key(COL_NAME);

- 如果想修改一列为主键，则需要先删除原来的主键：

alter table TABLE_NAME drop primary key;

再重新添加主键：

- alter table TABLE_NAME add primary key(COL_NAME);

<hr />
