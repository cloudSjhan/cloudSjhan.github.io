---
title: golang的多态实现
tags: [golang]
copyright: true
date: 2019-03-18 15:57:27
permalink:
categories: golang
description: golang的多态实现与原理
image: https://ss1.bdstatic.com/70cFvXSh_Q1YnxGkpoWK1HF6hhy/it/u=2486905288,2339571607&fm=26&gp=0.jpg
---
<p class="description"></p>

<img src="https://" alt="" style="width:100%" />

<!-- more -->

### 多态（polymorphism）

多态是接口的一个关键功能和Go语言的一个重要特性。

当非接口类型`T`的一个值`t`被包裹在接口类型`I`的一个接口值`i`中，通过`i`调用接口类型`I`指定的一个方法时，事实上为非接口类型`T`声明的对应方法将通过非接口值`t`被调用。 换句话说，**调用一个接口值的方法实际上将调用此接口值的动态值的对应方法**。 比如，当方法`i.m`被调用时，其实被调用的是方法`t.m`。 一个接口值可以通过包裹不同动态类型的动态值来表现出各种不同的行为，这称为多态。

当方法`i.m`被调用时，`i`存储的实现关系信息的方法表中的方法`t.m`将被找到并被调用。 此方法表是一个切片，所以此寻找过程只不过是一个切片元素访问操作，不会消耗很多时间。

注意，在nil接口值上调用方法将产生一个恐慌，因为没有具体的方法可被调用。

一个例子：

```go
package main

import "fmt"

type Filter interface {
	About() string
	Process([]int) []int
}

// UniqueFilter用来删除重复的数字。
type UniqueFilter struct{}
func (UniqueFilter) About() string {
	return "删除重复的数字"
}
func (UniqueFilter) Process(inputs []int) []int {
	outs := make([]int, 0, len(inputs))
	pusheds := make(map[int]bool)
	for _, n := range inputs {
		if !pusheds[n] {
			pusheds[n] = true
			outs = append(outs, n)
		}
	}
	return outs
}

// MultipleFilter用来只保留某个整数的倍数数字。
type MultipleFilter int
func (mf MultipleFilter) About() string {
	return fmt.Sprintf("保留%v的倍数", mf)
}
func (mf MultipleFilter) Process(inputs []int) []int {
	var outs = make([]int, 0, len(inputs))
	for _, n := range inputs {
		if n % int(mf) == 0 {
			outs = append(outs, n)
		}
	}
	return outs
}

// 在多态特性的帮助下，只需要一个filteAndPrint函数。
func filteAndPrint(fltr Filter, unfiltered []int) []int {
	// 在fltr参数上调用方法其实是调用fltr的动态值的方法。
	filtered := fltr.Process(unfiltered)
	fmt.Println(fltr.About() + ":\n\t", filtered)
	return filtered
}

func main() {
	numbers := []int{12, 7, 21, 12, 12, 26, 25, 21, 30}
	fmt.Println("过滤之前：\n\t", numbers)

	// 三个非接口值被包裹在一个Filter切片的三个接口元素中。
	filters := []Filter{
		UniqueFilter{},
		MultipleFilter(2),
		MultipleFilter(3),
	}

	// 每个切片元素将被赋值给类型为Filter的循环变量fltr。
	// 每个元素中的动态值也将被同时复制并被包裹在循环变量fltr中。
	for _, fltr := range filters {
		numbers = filteAndPrint(fltr, numbers)
	}
}
```

输出结果：

```
过滤之前：
	 [12 7 21 12 12 26 25 21 30]
删除重复的数字:
	 [12 7 21 26 25 30]
保留2的倍数:
	 [12 26 30]
保留3的倍数:
	 [12 30]
```

在上面这个例子中，多态使得我们不必为每个过滤器类型写一个单独的`filteAndPrint`函数。

除了上述这个好处，多态也使得一个代码包的开发者可以在此代码包中声明一个接口类型并声明一个拥有此接口类型参数的函数（或者方法），从而此代码包的一个用户可以在用户包中声明一个实现了此接口类型的用户类型，并且将此用户类型的值做为实参传递给此代码包中声明的函数（或者方法）的调用。 此代码包的开发者并不用关心一个用户类型具体是如何声明的，只要此用户类型满足此代码包中声明的接口类型规定的行为即可。

事实上，多态对于一个语言来说并非一个不可或缺的特性。我们可以通过其它途径来实现多态的作用。 但是，多态可以使得我们的代码更加简洁和优雅。

<hr />
