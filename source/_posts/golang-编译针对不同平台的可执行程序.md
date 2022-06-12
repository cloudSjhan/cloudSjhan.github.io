---
title: golang 编译针对不同平台的可执行程序
tags: [golang]
copyright: true
date: 2018-09-07 15:46:32
permalink:
categories: golang
description: 使用go build 编译同一套代码，在不同的平台运行
image:
---
<p class="description"></p>

<img src="https://" alt="" style="width:100%" />

<!-- more -->

```shell
Golang 支持在一个平台下生成另一个平台可执行程序的交叉编译功能。


Mac下编译Linux, Windows平台的64位可执行程序：


CGO_ENABLED=0 GOOS=linux GOARCH=amd64 go build test.go
CGO_ENABLED=0 GOOS=windows GOARCH=amd64 go build test.go
Linux下编译Mac, Windows平台的64位可执行程序：


CGO_ENABLED=0 GOOS=darwin GOARCH=amd64 go build test.go
CGO_ENABLED=0 GOOS=windows GOARCH=amd64 go build test.go
Windows下编译Mac, Linux平台的64位可执行程序：


SET CGO_ENABLED=0
SET GOOS=darwin3
SET GOARCH=amd64
go build test.go


SET CGO_ENABLED=0
SET GOOS=linux
SET GOARCH=amd64
go build test.go
 


GOOS：目标可执行程序运行操作系统，支持 darwin，freebsd，linux，windows
GOARCH：目标可执行程序操作系统构架，包括 386，amd64，arm


Golang version 1.5以前版本在首次交叉编译时还需要配置交叉编译环境：


CGO_ENABLED=0 GOOS=linux GOARCH=amd64 ./make.bash
CGO_ENABLED=0 GOOS=windows GOARCH=amd64 ./make.bash
```



<hr />
