---
title: Fuzzing is Beta Ready
tags: [Go]
copyright: true
date: 2021-06-15 10:12:20
permalink:
categories: Go
description: Golang Fuzzing is beta ready
image: https://static001.geekbang.org/resource/image/c0/c1/c055b48fff888567feea9be0eda843c1.jpg
---
<p class="description"></p>

<img src="https://" alt="" style="width:100%" />

<!-- more -->

近日 Golang 团队宣布，go 原生支持的模糊测试 (fuzzing) 的 beta 版本在 development 分支上已经可以使用了。开发人员可以使用该特性为项目构建 fuzzing 测试。

那么先来看下什么是 Fuzzing 呢？

### **什么是 Fuzzing？**

Fuzz 本意是“羽毛、细小的毛发、使模糊、变得模糊”，后来用在软件测试领域，中文一般指“模糊测试”，英文有的叫“Fuzzing”，有的叫“Fuzz Testing”。本文用 fuzzing 表示模糊测试。

Fuzzing 技术可以追溯到 1950 年，当时计算机的数据主要保存在打孔卡片上，计算机程序读取这些卡片的数据进行计算和输出。如果碰到一些垃圾卡片或一些废弃不适配的卡片，对应的计算机程序就可能产生错误和异常甚至崩溃，这样，Bug 就产生了。所以，Fuzzing 技术并不是什么新鲜技术，而是随着计算机的产生一起产生的古老的测试技术。

Fuzzing 技术是一种基于黑盒（或灰盒）的测试技术，通过自动化生成并执行大量的随机测试用例来发现产品或协议的未知漏洞。随着计算机的发展，Fuzzing 技术也在不断发展。

### **Fuzzing有用么？**

Fuzzing 是模糊测试，顾名思义，意味着测试用例是不确定的、模糊的。

计算机是精确的科学和技术，测试技术应该也是一样的，有什么的输入，对应什么样的输出，都应该是明确的，怎么会有模糊不确定的用例呢？这些不确定的测试用例具体会有什么作用呢？

为什么会有不确定的测试用例，我想主要的原因是下面几点：

1、我们无法穷举所有的输入作为测试用例。我们编写测试用例的时候，一般考虑正向测试、反向测试、边界值、超长、超短等一些常见的场景，但我们是没有办法把所有的输入都遍历进行测试的。

2、我们无法想到所有可能的异常场景。由于人类脑力的限制，我们没有办法想到所有可能的异常组合，尤其是现在的软件越来越多的依赖操作系统、中间件、第三方组件，这些系统里的bug或者组合后形成的 bug，是我们某个项目组的开发人员、测试人员无法预知的。

3、Fuzzing 软件也同样无法遍历所有的异常场景。随着现在软件越来越复杂，可选的输入可以认为有无限个组合，所以即使是使用软件来遍历也是不可能实现的，否则你的版本可能就永远也发布不了。Fuzzing 技术本质是依靠随机函数生成随机测试用例来进行测试验证，所以是不确定的。

### 这些不确定的测试用例会起到我们想要的测试结果么？能发现真正的 Bug 么？

1、Fuzzing 技术首先是一种自动化技术，即软件自动执行相对随机的测试用例。因为是依靠计算机软件自动执行，所以测试效率相对人来讲远远高出几个数量级。比如，一个优秀的测试人员，一天能执行的测试用例数量最多也就是几十个，很难达到 100 个。而 Fuzzing 工具可能几分钟就可以轻松执行上百个测试用例。

2、Fuzzing 技术本质是依赖随机函数生成随机测试用例，随机性意味着不重复、不可预测，可能有意想不到的输入和结果。

3、根据概率论里面的“大数定律”，只要我们重复的次数够多、随机性够强，那些概率极低的偶然事件就必然会出现。Fuzzing 技术就是大数定律的典范应用，足够多的测试用例和随机性，就可以让那些隐藏的很深很难出现的Bug成为必然现象。

目前，Fuzzing 技术已经是软件测试、漏洞挖掘领域的最有效的手段之一。Fuzzing 技术特别适合用于发现 0 Day 漏洞，也是众多黑客或黑帽子发现软件漏洞的首选技术。Fuzzing 虽然不能直接达到入侵的效果，但是 Fuzzing 非常容易找到软件或系统的漏洞，以此为突破口深入分析，就更容易找到入侵路径，这就是黑客喜欢 Fuzzing 技术的原因。

想要了解更多特性可以参考 golang fuzzing 的设计草案  [draft-fuzzing]( [https://go.googlesource.com/proposal/+/master/design/draft-fuzzing.md](https://go.googlesource.com/proposal/+/master/design/draft-fuzzing.md))

### 快速上手

```jsx
$ go get golang.org/dl/gotip
$ gotip download dev.fuzz
```

这将从 Dev.Fuzz 开发分支中构建 Go Toolchain，等将来代码合并到 Master 分支后就不需要自己构建了。 

```jsx
gotip test -fuzz=FuzzFoo
```

DEV.Fuzz 分支中会有正在进行的开发和错误修复，需要每次运行 `gotip download dev.fuzz` 以拉取最新的代码。

为了保障代码兼容性，在提交包含模糊测试的源文件时使用 `gofuzzbeta` 构建 tag。 默认情况下在Dev.Fuzz分支中的构建时已经启用此 tag。

```jsx
// +build gofuzzbeta
```

### Fuzz 测试样例

Fuzz 测试与普通的单元测试类似，需要在 `*_test.go` 中定义 `Fuzzxxx` 函数。 此函数必须传递`*testing.f` 参数，就像 `* testing.t` 参数传递给 testxxx 函数一样。

下面是测试 `net/url` 的 fuzzing target 的示例。

```go
// +build gofuzzbeta

package fuzz

import (
    "net/url"
    "reflect"
    "testing"
)

func FuzzParseQuery(f *testing.F) {
    f.Add("x=1&y=2")
    f.Fuzz(func(t *testing.T, queryStr string) {
        query, err := url.ParseQuery(queryStr)
        if err != nil {
            t.Skip()
        }
        queryStr2 := query.Encode()
        query2, err := url.ParseQuery(queryStr2)
        if err != nil {
            t.Fatalf("ParseQuery failed to decode a valid encoded query %s: %v", queryStr2, err)
        }
        if !reflect.DeepEqual(query, query2) {
            t.Errorf("ParseQuery gave different query after being encoded\nbefore: %v\nafter: %v", query, query2)
        }
    })
}
```

可以通过下面的命令使用 Go Doc 阅读有关 Fuzz API 的更多信息。

```go
gotip doc testing
gotip doc testing.F
gotip doc testing.F.Add
gotip doc testing.F.Fuzz
```

### 须知

1. 模糊测试会消耗大量内存，并在运行时可能会影响机器的性能。 `go test -fuzz` 默认值以并行运行 `$ gomaxProcs` 进程中的模糊。 可以在运行 `Go Test` 时加上 `--parallel` 来限制并发使用 cpu 的数量。 
2. 要注意目前没有限制写入 fuzzing 缓存的文件数或总字节数，因此它可能占据大量存储（即几个GB），可以通过运行 `gotip clean -fuzzcache` 来清除模糊测试后的缓存。

### 最后

这个特性并不会出现在即将发布的 Go1.17 中，但是计划在未来的某个 Go 版本中正式 release 这个特性。官方希望这个特性能够帮助 Go 开发人员开始编写 fuzzing 测试，并提供关于 fuzzing 的设计的反馈，为未来该功能的代码正式合并 master 做准备。

                                                                       Happy fuzzing!
<hr />
