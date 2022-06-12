---
title: Mysql无法连接[MySql Host is blocked because of many connection errors]
tags: [mysql]
copyright: true
date: 2018-09-01 13:20:54
permalink:
categories: MySQL
description: mysql 出现[MySql Host is blocked because of many connection errors]的错误
image:
---
<p class="description"></p>

<!-- more -->

* 测试环境，发现数据库（MySQL数据库）无法登录，报错如下：

  Host is blocked because of many connection errors; unblock with 'mysqladmin flush-hosts'

* 解决方案：使用mysqladmin flush-hosts 命令清理一下hosts文件（不知道mysqladmin在哪个目录下可以使用命令查找：whereis mysqladmin）；

* 登录到MySQL数据库中，mysql -uroot -h host -p

* 执行

  ```sql
  mysqladmin flush-hosts
  ```

  问题解决。

<hr />
