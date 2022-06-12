---
title: Why is there no Goroutine ID?
tags: [golang]
copyright: true
date: 2020-11-09 11:32:25
permalink:
categories: golang
description: Why is there no Goroutine ID?
image: https://static001.geekbang.org/resource/image/66/87/668c8c998ceb1c3ec96031a514ace387.jpg
---
<p class="description"></p>

<img src="https://" alt="" style="width:100%" />

<!-- more -->

在C/C++/Java等语言中，我们可以直接获取Thread Id，然后通过映射Thread Id和二级调度Task Id的关系，可以在日志中打印当前的TaskId，即用户不感知Task Id的打印，适配层统一封装，这使得多线程并发的日志的查看或过滤变得非常容易。

Goroutine 是 Golang 中轻量级线程的实现，由 Go Runtime 管理。Golang 在语言级别支持轻量级线程，叫协程。Golang 标准库提供的所有系统调用操作（当然也包括所有同步 IO 操作），都会出让 CPU 给其他 Goroutine。这让事情变得非常简单，让轻量级线程的切换管理不依赖于系统的线程和进程，也不依赖于 CPU 的核心数量。

官方曾经使用 `IIRC` 来暴露 `GoId` ，但是自从 go1.4 版本以后，Goroutine Id 无法直接从 Go Runtime 获取了。

禁用的原因有以下两点：

1. 当 Goroutine 关闭时，Goroutine 的 local storage 并不会立即被垃圾回收，这意味你只能拿到当前的 Goroutine 的 ID，但是不能正确拿到系统中所有正在运行的 GoId 列表。
2. 你只能获取你写的代码生成的 Goroutine ID，但是通常你不能确保所有的标准库以及第三方代码库都做了这些工作。

这样就很难对高并发日志进行查看和过滤。尽管在日志中可以使用业务本身的 ID ，但是在很多函数中仅仅为了打印而增加一些参数 ID 会让代码看起来没有那么优雅。

可以到 https://golang.org/doc/faq#no_Goroutine_id 中看到官方对这个问题的详细解释，原文不长所以放在文章里。

------------------------------------

### Why is there no Goroutine ID?

Goroutines do not have names; they are just anonymous workers. They expose no unique identifier, name, or data structure to the programmer. Some people are surprised by this, expecting the `go` statement to return some item that can be used to access and control the Goroutine later.

The fundamental reason Goroutines are anonymous is so that the full Go language is available when programming concurrent code. By contrast, the usage patterns that develop when threads and Goroutines are named can restrict what a library using them can do.

Here is an illustration of the difficulties. Once one names a Goroutine and constructs a model around it, it becomes special, and one is tempted to associate all computation with that Goroutine, ignoring the possibility of using multiple, possibly shared Goroutines for the processing. If the `net/http` package associated per-request state with a Goroutine, clients would be unable to use more Goroutines when serving a request.

Moreover, experience with libraries such as those for graphics systems that require all processing to occur on the "main thread" has shown how awkward and limiting the approach can be when deployed in a concurrent language. The very existence of a special thread or Goroutine forces the programmer to distort the program to avoid crashes and other problems caused by inadvertently operating on the wrong thread.

For those cases where a particular Goroutine is truly special, the language provides features such as channels that can be used in flexible ways to interact with it.
<hr />
