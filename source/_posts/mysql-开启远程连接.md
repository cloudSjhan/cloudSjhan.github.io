---
title: mysql 开启远程连接
tags: [mysql]
copyright: true
date: 2018-08-29 11:17:20
permalink:
categories: MySQL
description: 不管是在测试还是开发中，MySQL经常需要开启远程连接功能
image: 
---
<p class="description"></p>

<!-- more -->

- 背景： 建站的时候会出现数据库和网站是不同的ip，就需要开启MySQL的远程连接服务，但是MySQL由于安全原因，默认设置是不允许远程只能本地连接，要开启远程连接就需要修改某些配置文件。

### 按照下面的步骤，开启MySQL的远程连接

* 进入数据库cmd

  ```shell
  mysql -uroot -h host -p
  Enter password:***
  ```

* 连接到默认mysql数据库

  ```shell
  show databases;
  
  use mysql;
  ```

* 配置

  ```shell
  Grant all privileges on *.* to 'root'@'host' identified by 'password' with grant option;
  ```

  host表示你远程连接数据库设备的ip地址（如果你想让所有机器都能远程连接，host改为‘%’，**不推荐这样使用**），password表示MySQL的root用户密码

* 刷新or重启MySQL

  ```shell
  mysql> flush privileges;
  ```

* 最后非常重要的一点

  ```shell
  vim /etc/vim /etc/mysql/my.cnf
  屏蔽bing-server 127.0.0.0
  #bing-server 127.0.0.0
  ```

* 完成，可以远程连接你的数据库了

<hr />
