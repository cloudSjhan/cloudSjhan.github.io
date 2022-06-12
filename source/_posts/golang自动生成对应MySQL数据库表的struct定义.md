---
title: golang自动生成对应MySQL数据库表的struct定义
tags: [golang]
copyright: true
date: 2020-01-01 22:37:14
permalink:
categories: golang
description: golang自动生成对应MySQL数据库表的struct定义
image: https://static001.geekbang.org/resource/image/93/b7/931d3c1dbd056aa39041a209e79fe0b7.jpg
---
<p class="description"></p>
<img src="https://" alt="" style="width:100%" />

<!-- more -->

在golang的开发过程中，当我们使用orm的时候，常常需要将数据库表对应到golang的一个struct，这些struct会携带orm对应的`tag`,就像下面的struct定义一样：

```go
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

这是gorm对应的数据库表的struct映射，即使数据表的字段不多，如果是手动写起来也是一些重复性的工作。像MySQL这种关系型数据库，我们一般会用orm去操作数据库，于是就想，mysql的数据表能不能来自动生成golang 的struct定义。我们知道mysql有个自带的数据库`information_schema`，有一张表`COLUMNS`，它的字段包含数据库名、表名、字段名、字段类型等，我们可以利用这个表的数据，把对应的表的字段信息读取出来，然后再根据golang的语法规则，生成对应的struct。
	调研了一下目前有一些命令行工具像 db2struct等，感觉用起来比较繁琐，在想能不能提供一个开箱即用的环境，提供web界面，我们只需要填写数据库信息，就可以一键生成对应的ORM的struct，于是就诞生了这个项目：https://github.com/hantmac/fuckdb

如果你的数据库在本地，那么只需要执行 `docker-compose up -d`，访问`localhost:8088`，你就会得到下面的界面：

![](https://tva1.sinaimg.cn/large/006tNbRwgy1g9w1ru6tl4j31wb0u0aft.jpg)

如果你的数据库在内网服务器上，你需要先修改后端接口的ip:port,然后重新build Docker镜像，push到自己的镜像仓库，然后修改docker-compose.yaml，再执行`docker-compose up -d`。修改的位置是：`fuckdb/frontend/src/config/index.js`.

![](https://tva1.sinaimg.cn/large/006tNbRwgy1gahgtkjlraj31000hkada.jpg)

只需要填入数据库相关信息，以及你想得到的golang代码的`package name`、`struct name`,然后点击生成，就可以得到gorm对应的结构体映射：

![](https://tva1.sinaimg.cn/large/006tNbRwly1g9w531osobj31u90u0jzq.jpg)

你只需要拷贝你的代码到项目中即可。我们都知道golang的struct的tag有很多功能，这里也提供了很多tag的可选项，比如`json`,`xml`等，后面会曾加更多的tag可选项支持。

![](http://static.vue2.net/db.gif)

上面的GIF展示了增加了缓存功能的版本，记忆你之前填写过的数据库信息，省去了大量重复的操作，你不用再填写繁琐的数据库名，表名，等等，只需一键，就可以得到对应的代码，是不是很方便啊。ps:目前数据库信息没有做加密，所以不方便放到公网上使用，仅限于内网，后面会进行相应的开发支持。目前这个工具在我们组内已经开始使用，反馈比较好，节省了很多重复的工作，尤其是在开发的时候用到同一个库的多张表，很快就可以完成数据库表->strcut的映射。

欢迎试用&反馈&Contribute。代码地址：https://github.com/hantmac/fuckdb

<hr />
