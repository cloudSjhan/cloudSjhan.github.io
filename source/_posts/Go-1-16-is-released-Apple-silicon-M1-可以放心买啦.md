---
title: 'Go 1.16 is released, Apple silicon M1 可以放心买啦'
tags: [golang]
copyright: true
date: 2021-02-20 15:32:51
permalink:
categories: golang
description: Go 1.16 is released, Apple silicon M1 可以放心买啦'
image: https://static001.geekbang.org/resource/image/da/89/dad54cea1adc9afb20600c916931be89.jpg
---
<p class="description"></p>

<img src="https://" alt="" style="width:100%" />

<!-- more -->


![https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/e6cc0de3ef1945649039cabc8c7e1580~tplv-k3u1fbpfcp-zoom-1.image](https://tva1.sinaimg.cn/large/008eGmZEgy1gnqdzsw78rj31gk0rgtgs.jpg)

2021 年 2 月 16 日，春节假期的最后一天，Go 官方终于将 1.16 版本 released。

下面简要介绍一下 1.16 版本最重要的一些特性：

### 核心库加入了新的成员 - embed

package `embed` 可访问正在运行的 Go 程序中所嵌入的静态文件。

使用 `embed` 可以使用 `// go：embed` 指令在编译时从包目录或子目录读取的文件的内容并使用它们。

例如，以下三种方法可以嵌入名为 `hello.txt` 的文件，然后在运行时打印其内容。

- 将一个文件嵌入到字符串中

```go
import _ "embed"

//go:embed hello.txt
var s string
print(s)
```

- 将一个文件嵌入 []byte

```go
import _ "embed"

//go:embed hello.txt
var b []byte
print(string(b))
```

- 将一个或多个文件嵌入到文件系统中

```go
import "embed"

//go:embed hello.txt
var f embed.FS
data, _ := f.ReadFile("hello.txt")
print(string(data))
```

这种将静态文件在编译时嵌入可执行文件的方式，在极大地提高了 go 访问静态文件的灵活性的同时，也能提高了敏感配置文件的安全性。更大胆一点，是不是在前端领域，golang 也能插一脚了？

### 增加对 Apple silicon ARM 64 架构的支持

Go 1.16 还添加了macOS ARM64 支持（也称为Apple silicon）。 自 Apple 宣布其新的 ARM64 架构以来，go team 一直在与他们紧密合作以确保 Go 得到完全的支持。一直在 观望 M1 的开发者这下可以放心去买新的 Mac 啦。

### 默认开启 Go modules

Go 1.16 默认使用 Go modules。因为根据 go team 的 2020 Go 开发人员调查，现在有96％的 Go 开发人员已经在使用 Go modules了。

### 其他的性能改善与提高

最后，还有许多其他改进和 bug fix，比如构建速度提高了25％，内存使用量减少了15％。 有关更改的完整列表以及有关上述改进的更多信息，可以参考 [Go 1.16发行说明]([https://golang.org/doc/go1.16](https://golang.org/doc/go1.16))。

以上就是 Go 1.16 带来的新特性，有开发者调侃到 “最大的特性就是离泛型的版本号更近了（狗头）”哈哈哈。

<hr />
