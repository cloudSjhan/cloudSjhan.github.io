---
title: 'fuckdb Lite, 帮助你更快地生成go struct代码'
tags: [go]
copyright: true
date: 2020-04-06 09:07:07
permalink:
categories: go
description: fuckdb Lite, 帮助你更快地生成go struct代码
image: https://tva1.sinaimg.cn/large/00831rSTgy1gdjs0oe8skj31fi0mq1iz.jpg
---
<p class="description"></p>

<img src="https://" alt="" style="width:100%" />

<!-- more -->

## 前言&背景
在golang的开发过程中，当我们使用orm的时候，常常需要将数据库表对应到golang的一个struct，这些struct会携带orm对应的`tag`，就像下面的struct定义一样：
```
type InsInfo struct {
  Connections  string    `gorm:"column:connections"`
  CPU          int       `gorm:"column:cpu"`
  CreateTime   time.Time `gorm:"column:create_time"`
  Env          int       `gorm:"column:env"`
  ID           int64     `gorm:"column:id;primary_key"`
  IP           string    `gorm:"column:ip"`
  Organization string    `gorm:"column:organization"`
  Pass         string    `gorm:"column:pass"`
  Port         string    `gorm:"column:port"`
  RegionId     string    `gorm:"column:regionid"`
  ServerIP     string    `gorm:"column:server_ip"`
  Status       int       `gorm:"column:status"`
  Type         string    `gorm:"column:type"`
  UUID         string    `gorm:"column:uuid"`
}
```

这是gorm对应的数据库表的struct映射，即使数据表的字段不多，如果是手动写起来也是一些重复性的工作。像MySQL这种关系型数据库，我们一般会用orm去操作数据库，于是就想mysql的数据表能不能来自动生成golang 的struct定义 ，减少重复性的开发工作（早点下班）。

## 现状
调研了一下目前有一些工具，比如chrome插件SQL2Struct(一款根据sql语句自动生成golang结构体的chrome插件)，感觉用起来比较繁琐，每次需要进入数据库，执行SQL语句拿到建表语句copy到浏览器中，才能使用。在想能不能提供一个开箱即用的环境，提供web界面，我们只需要填写数据库信息，就可以一键生成对应的ORM的struct，于是就诞生了这个项目：https://github.com/hantmac/fuckdb

## 原理
mysql有个自带的数据库`information_schema`，有一张表`COLUMNS`，它的字段包含数据库名、表名、字段名、字段类型等，我们可以利用这个表的数据，把对应的表的字段信息读取出来，然后再根据golang的语法规则，生成对应的struct。具体不详细展开了，感兴趣的可以去看下源码。

## Web 版
### 连接本地数据库
如果你的数据库在本地，那么只需要执行 `docker-compose up -d`，
访问`localhost:8000`，你就会得到下面的界面：


![](https://user-gold-cdn.xitu.io/2020/4/6/1714cf8416f8ebde?w=3324&h=892&f=png&s=151985)
### 服务器上的数据库
如果你的数据库在内网服务器上，你需要先修改后端接口的ip:port,然后重新build Docker镜像，push到自己的镜像仓库，然后修改docker-compose.yaml，再执行`docker-compose up -d`。修改的位置是：`fuckdb/frontend/src/config/index.js`.
```
let APIdb2struct

if(process.env.NODE_ENV === "development"){
  APIdb2struct = "http://0.0.0.0:8000" //修改为部署服务器的ip
}else{
  APIdb2struct = "http://0.0.0.0:8000" //修改为部署服务器的ip
}

export default {
  APIdb2struct
}

```

只需要填入数据库相关信息，以及你想得到的golang代码的`package name`、`struct name`,然后点击生成，就可以得到gorm对应的结构体映射。

在你的项目项目中只要 Ctrl+C&Ctrl+V 即可。我们知道golang的struct的tag有很多功能，这里也提供了很多tag的可选项，比如`json`,`xml`等，后面会增加更多的tag可选项支持。

web版的特色功能是数据库信息缓存功能，能够记忆你之前填写过的数据库信息，省去了大量重复的操作，你不用再填写繁琐的数据库名，表名，只需一键，就可以得到对应的代码，配合附带json-to-go插件(https://github.com/mholt/json-to-go)，开发效率得到极速提升。目前这个工具在我们组内已经开始使用，反馈比较好，节省了很多重复的工作，尤其是在开发的时候用到同一个库的多张表，很快就可以完成数据库表->strcut的映射。

**来看一段演示视频。**

![](https://user-gold-cdn.xitu.io/2020/4/6/1714cf9e25576889?w=1920&h=1080&f=gif&s=765691)

## 插曲
前几天有同学找上门，说fuckdb的web版部署后无法使用，解决了半天也没能让用户部署起来，反馈过来还是感觉部署有些复杂。反思了一下，对于一个工具化的软件，有些用户并不想做一些复杂的部署流程或者不熟悉部署操作，可能就是想暂时使用一下，所以应该让工具更加轻量化，更加开箱即用，于是连夜写了一个fuckdb lite, 更容易上手使用，更方便的安装流程，1分钟拿到你想要的代码。

## fuckdb Lite
### 原理
基于 cobra(https://github.com/spf13/cobra)，核心代码继承web版。
### 安装

听取用户反馈，安装流程极简化，Mac用户可以直接brew install 安装

```
brew tap hantmac/tap && brew install fuckdb
```
- Linux用户:
`curl https://github.com/hantmac/fuckdb/releases/download/v0.2/fuckdb_linux.tar.gz` 下载、解压、安装
- windows用户emmm, 就去GitHub的release手动下载吧

### 使用
```
fuckdb --help
From mysql schema generate golang struct with gorm, json tag

Usage:
  fuckdb [command]

Available Commands:
  generate    use `fuckdb generate` to generate fuckdb.json
  go          fuckdb go to generate golang struct with gorm and json tag
  help        Help about any command

Flags:
  -h, --help   help for fuckdb

Use "fuckdb [command] --help" for more information about a command.
```
目前提供了两个主要命令，`fuckdb generate` 和 `fuckdb go`，我们依次来看。

```
fuckdb generate
```
生成一个存储MySQL信息的`fuckdb.json`文件,
编辑 fuckdb.json  ，填写你的MySQL信息，该文件可复用，简单修改表名即可。
```
{
  "db": {
    "host": "localhost",
    "port": 3306,
    "password": "password",
    "user": "root",
    "table": "tableName",
    "database": "example",
    "packageName": "packageName",
    "structName": "structName",
    "jsonAnnotation": true,
    "gormAnnotation": true
  }
}
​
```

修改完文件后，就完成了准备工作，go go go!

​执行
```
fuckdb go
```

![](https://user-gold-cdn.xitu.io/2020/4/6/1714cfdc08b200b9?w=1238&h=528&f=png&s=619750)
**Enjoy Your Code！**

来看一段演示操作(说好一分钟拿到代码，绝不超1秒)


![](https://user-gold-cdn.xitu.io/2020/4/6/1714cfdf3764125e?w=1736&h=1080&f=gif&s=675007)


比之前的web版的安装简直方便了太多，妈妈再也不用担心我加班啦。

ps:  fuckdb.json文件必须在操作目录下。

欢迎试用&反馈&Contribute。代码地址：https://github.com/hantmac/fuckdb

------------------------------------------------------------------------------
                                    官方资讯*最新技术*独家解读


![](https://user-gold-cdn.xitu.io/2020/4/6/1714cfed71ef0758?w=227&h=227&f=jpeg&s=24842)



<hr />
