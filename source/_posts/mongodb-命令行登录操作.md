---
title: mongodb 命令行登录操作
tags: [MongoDB]
copyright: true
date: 2019-04-11 15:07:14
permalink:
categories: MongoDB
description: mongodb 命令行登录操作
image: https://static001.geekbang.org/resource/image/c0/c1/c055b48fff888567feea9be0eda843c1.jpg
---
<p class="description"></p>

<img src="https://" alt="" style="width:100%" />

<!-- more -->

## Mac OS下载安装及配置

使用brew可以直接搞定，简单方便！

```
brew install mongodb
```

安装完成后会提示启动的方式，安装位置等等信息。



![img](https:////upload-images.jianshu.io/upload_images/4218662-435c85a12eae4a61.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/958/format/webp)

安装完成

安装完成后，按照提示启动MongoDB：

```
mongod --config /usr/local/etc/mongod.conf
```

配置文件已修改fork=true，因此启动后将以后台进程方式存在。



![img](https:////upload-images.jianshu.io/upload_images/4218662-7842d5837475c137.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1000/format/webp)

MongoDB启动

终端输入mongo连接MongoDB数据库(默认连接本机的27017端口)

```
mongo
```



![img](https:////upload-images.jianshu.io/upload_images/4218662-8f368a1234b354cf.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1000/format/webp)

客户端连接

## 配置文件

MongoDB引入一个YAML-based格式的配置文件。
 现在的配置文件/usr/local/etc/mongod.conf显示如下：

```
# 日志
systemLog:
# 日志为文件
  destination: file
# 文件位置
  path: /usr/local/var/log/mongodb/mongo.log
# 是否追加
  logAppend: true
#进程
processManagement:
# 守护进程方式
  fork: true
storage:
  dbPath: /usr/local/var/mongodb
net:
# 绑定IP，默认127.0.0.1，只能本机访问
  bindIp: 127.0.0.1
# 端口
  port: 27017
```



## [CentOS 7 安装MongoDB详细步骤](https://segmentfault.com/a/1190000016877915)

创建`/etc/yum.repos.d/mongodb-org-4.0.repo`文件，编辑内容如下：

```
[mongodb-org-4.0]
name=MongoDB Repository
baseurl=https://repo.mongodb.org/yum/redhat/$releasever/mongodb-org/4.0/x86_64/
gpgcheck=1
enabled=1
gpgkey=https://www.mongodb.org/static/pgp/server-4.0.asc
```

运行以下命令安装最新版的mongodb：

```
sudo yum install -y mongodb-org
```

配置`mongod.conf`允许远程连接：

```
$ vim /etc/mongod.conf

# Listen to all ip address
bind_ip = 0.0.0.0
```

启动mongodb：

```
sudo service mongod start
```

创建管理员用户：

```
$ mongo
>use admin
 db.createUser(
  {
    user: "myUserAdmin",
    pwd: "abc123",
    roles: [ { role: "userAdminAnyDatabase", db: "admin" }, "readWriteAnyDatabase" ]
  }
 )
```

启用权限管理：

```
$ vim /etc/mongod.conf

#security 
security:
  authorization: enabled
```

重启mongodb：



```
sudo service mongod restart
```

###  Mongodb 用户验证登陆

启动带访问控制的 Mongodb
 新建终端

```
mongod --auth --port 27017 --dbpath /data/db1
```

现在有两种方式进行用户身份的验证
 第一种 （类似 MySql）
 客户端连接时，指定用户名，密码，db名称

```
mongo --port 27017 -u "adminUser" -p "adminPass" --authenticationDatabase "admin"
```

第二种
 客户端连接后，再进行验证

```
mongo --port 27017

use admin
db.auth("adminUser", "adminPass")

// 输出 1 表示验证成功
```

###  创建普通用户

过程类似创建管理员账户，只是 role 有所不同

```
use foo

db.createUser(
  {
    user: "simpleUser",
    pwd: "simplePass",
    roles: [ { role: "readWrite", db: "foo" },
             { role: "read", db: "bar" } ]
  }
)
```

现在我们有了一个普通用户
 用户名：simpleUser
 密码：simplePass
 权限：读写数据库 foo， 只读数据库 bar。

**注意**
 **NOTE**
 **WARN**
 `use foo`表示用户在 foo 库中创建，就一定要 foo 库验证身份，即用户的信息跟随随数据库。比如上述 simpleUser 虽然有 bar 库的读取权限，但是一定要先在 foo 库进行身份验证，直接访问会提示验证失败。

```
use foo
db.auth("simpleUser", "simplePass")

use bar
show collections
```

还有一点需要注意，如果 admin 库没有任何用户的话，即使在其他数据库中创建了用户，启用身份验证，默认的连接方式依然会有超级权限

<hr />
