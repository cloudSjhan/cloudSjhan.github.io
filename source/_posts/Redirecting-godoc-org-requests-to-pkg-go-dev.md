---
title: Redirecting godoc.org requests to pkg.go.dev
tags: [golang]
copyright: true
date: 2020-12-16 14:54:42
permalink:
categories: golang
description: Redirecting godoc.org requests to pkg.go.dev
image: https://tva1.sinaimg.cn/large/0081Kckwgy1glpqc0oq9aj31os0joad0.jpg
---
<p class="description"></p>

<img src="https://" alt="" style="width:100%" />

<!-- more -->

### 现状

随着 Go Module 的引入和 Go 生态系统的发展，pkg.go.dev 于 [2019 年](https://blog.golang.org/go.dev)启动，为开发人员提供了了一个查找 Go package 的地方，官方称之为 `center place`。 像 godoc.org 一样，pkg.go.dev 提供 Go 文档，但它也支持 Go Module、更好的搜索功能以及帮助 Go 用户找到正确软件包的指引。

正如官方在 [2020年1月分享](https://blog.golang.org/pkg.go.dev-2020) 的那样，官方的目标是最终将流量从 godoc.org 重定向到 pkg.go.dev 上的相应页面。 用户可以还可以选择将自己的请求从godoc.org 重定向到 pkg.go.dev。

今年官方收到了很多反馈，很多问题已经在 [pkgsite/godoc.org-redirect](https://github.com/golang/go/milestone/157?closed=1https://github.com/golang/go/milestone/157?closed=1) 和 [pkgsite/design-2020](https://github.com/golang/go/milestone/159?closed=1) 进行跟踪和解决。 用户的反馈意见支持对 pkg.go.dev 上的高频功能的改进，以及最近对pkg.go.dev 的重新设计都有很大的帮助。

### 下一步

一旦在 [pkgsite/godoc.org-redirect](https://github.com/golang/go/milestone/157) 里程碑中跟踪的工作完成, 官方就会将所有请求从 godoc.org 重定向到pkg.go.dev上的相应页面,这大概会在2021 年初开始。

官方鼓励大家现状就开始使用 pkg.go.dev, 可以通过访问 `godoc.org?redirect=on` 或单击任何`godoc.org` 页面右上角的 “Always use pkg.go.dev” 来实现。

### FAQs
- A: godoc.org 还可以继续使用吗？
  Q: YES! 我们会将所有访问 godoc.org 的请求重定向到 pkg.go.dev 上的相应页面，因此你所有的书签和链接依然有效。

- A: `golang/gddo` repo 将会如何处理？
  Q: repo 可以继续 fork 和使用，但是官方将对其标记为 `archived`。

- A: `api.godoc.org` 还能继续使用吗？
  Q: 此过渡不会对 api.godoc.org 产生影响。 在 pkg.go.dev 提供 API 之前，api.godoc.org 将继续为流量提供服务。 有关 pkg.go.dev 的 API 的更新，请参见 [issue 36785](https://golang.org/issue/36785)。

### Contributing

  [Pkg.go.dev](https://go.googlesource.com/pkgsite) 是一个开源项目。 如果你有兴趣为 pkg site 做出贡献，可以加入 Gophers Slack 上的 `#pkgsite` 频道以了解更多信息。

<hr />
