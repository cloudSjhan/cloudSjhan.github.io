---
title: beego注解路由中@Param的参数解释
tags: [beego,go]
copyright: true
date: 2019-01-03 16:56:19
permalink:
categories: go
description: beego注解路由中@Param的参数解释
image: https://static001.geekbang.org/resource/image/8c/db/8c181c2b9b0b840762eb1f393ca974db.jpg
---
<p class="description"></p>

<img src="https://" alt="" style="width:100%" />

<!-- more -->

beego注解路由的注释，我们可以把我们的注释分为以下类别：

- @Title

  接口的标题，用来标示唯一性，唯一，可选

  格式：之后跟一个描述字符串

- @Description

  接口的作用，用来描述接口的用途，唯一，可选

  格式：之后跟一个描述字符串

- [@Param](http://my.oschina.net/param)

  请求的参数，用来描述接受的参数，多个，可选

  格式：变量名 传输类型 类型 是否必须 描述

  传输类型：paht or body

  类型：

  变量名和描述是一个字符串

  是否必须：true 或者false

- - string
  - int
  - int64
  - 对象，这个地方大家写的时候需要注意，需要是相对于当前项目的路径.对象，例如`models.Object`表示`models`目录下的Object对象，这样bee在生成文档的时候会去扫描改对象并显示给用户改对象。
  - query 表示带在url串里面?aa=bb&cc=dd
  - form 表示使用表单递交数据
  - path 表示URL串中得字符，例如/user/{uid} 那么uid就是一个path类型的参数
  - body 表示使用raw body进行数据的传输
  - header 表示通过header进行数据的传输

- 
- [@Success](http://my.oschina.net/u/660711)

  成功返回的code和对象或者信息

  格式：code 对象类型 信息或者对象路径

  code：表示HTTP的标准status code，200 201等

  对象类型：{object}表示对象，其他默认都认为是字符类型，会显示第三个参数给用户，如果是{object}类型，那么就会去扫描改对象，并显示给用户

  对象路径和上面Param中得对象类型一样，使用路径.对象的方式来描述

- [@Failure](http://my.oschina.net/Failure)

  错误返回的信息，

  格式： code 信息

  code:同上Success

  错误信息：字符串描述信息

- [@router](http://my.oschina.net/Router)

  上面已经描述过支持两个参数，第一个是路由，第二个表示支持的HTTP方法

<hr />
