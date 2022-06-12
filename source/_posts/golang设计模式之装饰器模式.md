---
title: golang设计模式之装饰器模式
tags: [golang,设计模式]
copyright: true
date: 2019-11-30 15:21:30
permalink:
categories: golang
description: golang设计模式之装饰器模式
image: https://static001.geekbang.org/resource/image/46/75/46819da18e726a3d6d76069e049c4b75.jpg
---
<p class="description"></p>
<img src="https://" alt="" style="width:100%" />

<!-- more -->

###  装饰器

装饰器结构模式允许动态地扩展现有对象的功能，而不改变其内部结构。

装饰器提供了一种灵活的方法来扩展对象的功能。

### golang 实现

下面的LogDecorate用signature func（int）int修饰函数，该函数操作整数并添加输入/输出日志记录功能。

```golang
type Object func(int) int

func LogDecorate(fn Object) Object {
	return func(n int) int {
		log.Println("Starting the execution with the integer", n)

		result := fn(n)

		log.Println("Execution is completed with the result", result)

        return result
	}
}
```

### 如何使用

```golang
func Double(n int) int {
    return n*2
}
f := LogDecorate(Double)
f(5)
//参数为5，开始执行
//执行的结果是10
```

### 经验

- 与适配器模式不同，要修饰的对象是通过注入获得的。

- 装饰器不应更改对象的接口。

<hr />
