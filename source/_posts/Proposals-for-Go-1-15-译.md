---
title: Proposals for Go 1.15(译)
tags: [golang]
copyright: true
date: 2020-02-01 10:48:54
permalink:
categories: golang
description: 本文是Golang官方对于Go近期版本的现状总结以及Go 1.15版本的一些提案，原文发布于 The Go Blog
image:
---
<p class="description"></p>
<img src="https://" alt="" style="width:100%" />

<!-- more -->

## [Proposals for Go 1.15](https://blog.golang.org/go1.15-proposals)(译)

----

*本文是Golang官方对于Go近期版本的现状总结以及Go 1.15版本的一些提案，原文发布于 The Go Blog*

*Robert Griesemer, for the Go team*
		*28 January 2020*


## 现状

Go 1.14的RC1版本已经准备就绪，如果一切顺利，Go 1.14计划将于今年2月份发布。按照[Go 2,here we come!](https://blog.golang.org/go2-here-we-come)的规划，在我们的开发周期中又到了这样的一个时刻：考虑下一个版本中（暂定于几年8月份发布的Go 1.15），到底要提供哪些Golang本身或者Go lib中的新特性。

当前版本的Go的优化目标仍然围绕着软件包和版本管理，更好的错误处理支持以及泛型特性。Go Module 目前使用状态比较好，并且每天都在不断完善，对泛型的支持也有一些推进（今年晚些时候会有更多进展）。为了提供更好的错误处理机制，大约在7个月前我们提出了[try_proposal](https://golang.org/issue/32437)的提案，有少部分人支持该提案，但是大部分还是持强烈的反对态度，于是我们决定放弃这个想法。在此之后，也有许多建议，但它们有些都没有足够的说服力，有些还不如[try_proposal](https://golang.org/issue/32437)。因此，我们暂时没有进一步的计划去改善错误处理机制，也许未来会出现一些比较好的点子帮助我们解决这个问题。

## 提案

鉴于Go Module和泛型正在积极的推进中，且错误处理机制没有进一步的计划，那么还有哪些特性值得我们去关注呢？如果非要说有的话，有一些编程语言中经久不衰的特性，例如枚举类型和不可变类型的需求，但是这些idea还没有被Go团队深入地思考，所以Go团队也没有投入过多精力在其中，尤其是改进这些语言特性需要很大的成本。

Go team不希望在没有长期计划的情况下增添很多新的特性，在审核了大量可行的提案之后，Go 团队得出结论：这一次不进行重大更改。相反，我们会集中精力做一些新的 `go vet`支持和语言层面的一下小调整。我们选择了以下三个提案：

[#32479](https://golang.org/issue/32479) `go vet`中支持string(int) 转换的检测

我们原计划在即将发布的Go 1.14版本中实现该功能，但我们并没有解决这个问题，所以放在Go 1.15中解决。string(int) 转换在很早的Go版本中就已经引入了，但是有些特性（string(10)是"\n"，并不是"10"）会让Go新手迷惑，并且unicode/utf8包中也已经提供了该功能。自从[removing this coversion](https://golang.org/issue/3939)的提案不是一个向后兼容的改变，所以我们需要在`go vet`中提供string(int)的错误检测能力。

[#4483](https://golang.org/issue/4483) `go vet`中支持错误的x.(T)的类型断言

当前，Go允许任何类型断言x.(T)（以及相应的类型切换情况），其中x和T为接口类型。但是，如果x和T有相同名字的方法，但是签名不同，则分配给x的任何值也不能实现T（这种类型的断言在运行时总是失败，panic或evaluate to false）。因为我们在编译时就知道这一点，所以编译器会报告错。在这种情况下，编译报错不是向后兼容的，因此我们也将在`go vet`中添加对该情况的检查。

[#28591](https://golang.org/issue/28591) 使用常量字符串和索引进行常量求值的索引和切片表达式

当前，用一个或多个常量索引对常量字符串进行索引或切片会分别产生一个非常量字节或字符串值。 但是，如果所有操作数都是常量，则编译器可以对这些表达式进行常量求值，并生成常量（可能是无类型的）结果。 这是完全向后兼容的更改，我们将对开发规范和编译器进行必要的调整。

## 时间线

Go team认为这三个提案应该是没有什么争议的，但是人非圣贤孰能无过，因此我们计划在Go 1.15的发布周期开发(Go 1.14发布时或之后)开始采纳开发者对该三个提案的建议，以便有足够的时间收集反馈。在[proposal evalution process](https://blog.golang.org/go2-here-we-come)，最终的提案将在2020年5月初决定。

## One more thing...

我们收到了大量的语言特性修正的提案([issue label LanguageChange](https://github.com/golang/go/labels/LanguageChange))，但是我们没有时间去仔细地彻底地评估。举个栗子，单是关于错误处理机制的提案，就有57个issue，到目前为止还有5个处于open的状态。由于进行语言更改的成本，无论大小如何，对于开发者来说其实都很高，而且收益往往很小，因此我们必须谨慎行事。由于大多数提案反馈的人数很少，这些提案都会遭到拒绝。但是有些开发者花费了大量时间去写提案的细节，到头来还是被拒；另一方面，由于总体提案流程比较简单，因此提交一些无关紧要的语言变更提案非常容易，从而给审核委员会带来大量不必要的工作，也有可能埋没比较好的提案。为了解决这个问题我们新增了一个关于Go提案的[问卷](https://github.com/golang/proposal/blob/master/go2-language-changes.md)：填写该模板将有助于审核者更有效地评估提案，因为他们无需尝试自己回答这些问题。 它能从一开始就设定一些限制，从而为提议者提供更好的指导。 这是一个实验性的尝试，我们会根据需要随着时间的推移进行完善。

感谢您帮助我们改善Go的体验！



--------

原文地址：https://blog.golang.org/go1.15-proposals

<hr />
