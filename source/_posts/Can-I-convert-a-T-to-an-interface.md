---
title: 'Can I convert a []T to an []interface{}?'
tags: [golang]
copyright: true
date: 2020-12-09 20:33:50
permalink:
categories: golang
description: Can I convert a []T to an []interface{}?
image: https://static001.geekbang.org/resource/image/0f/76/0f0963bc50f97aa1171fa81ae6e96776.jpg
---
<p class="description"></p>

<img src="https://" alt="" style="width:100%" />

<!-- more -->


### Can I convert a []T to an []interface{}?

众所周知，在 Golang 中，interface 是一种抽象类型，相对于抽象类型的是具体类型（concrete type）：int，string。`interface{}` 是一个空的 interface 类型，一个类型如果实现了一个 interface 的所有方法就说该类型实现了这个 interface，空的 interface 没有方法，所以可以认为所有的类型都实现了 `interface{}`。如果定义一个函数参数是 `interface{}` 类型，这个函数应该可以接受任何类型作为它的参数。

```go
func changeType(v interface{}){    
}
```

既然空的 interface 可以接受任何类型的参数，那么一个 `interface{}`类型的 slice 是不是就可以接受任何类型的 slice ?看下面的代码：

```go
func traverseSlice(vals []interface{}) { 
	for _, val := range vals {
		fmt.Println(val)
	}
}

func main(){
	nums := []string{"1", "2", "3"}
	traverseSlice(nums)
}
```

不是说空的 interface 可以是任何类型吗？为何报了下面的错误？

![image-20201209202110198](https://tva1.sinaimg.cn/large/0081Kckwgy1glhvf447lfj30ys04w0tk.jpg)

这个错误说明 go 没有自动把 `[]string` 转换成 `[]interface{}` ，所以出错了。**go 不会对 类型是`interface{}` 的 slice 进行转换** 。为什么 go 不帮我们自动转换? 在 golang 官方 blog 的 FAQ 中找到了对这个问题的解释，由于`interface{}` 会占用两部分存储空间，一个是自身的 methods 数据，一个是指向其存储值的指针，也就是 interface 变量存储的值，因而 slice []interface{} 其长度是固定的`N*2`，但是 []T 的长度是`N*sizeof(T)`，用官方的解释就是两种类型在内存中的表现是不一样的。那该如何去转换呢？可以按照单个元素来转换：

```go
import "fmt"

func main()  {
	t := []int{1, 2, 3, 4}
	s := make([]interface{}, len(t))

	for i, v := range t {
		s[i] = v
	}

	fmt.Println(s)
}

```

![image-20201209203108747](https://tva1.sinaimg.cn/large/0081Kckwgy1glhvpfhteij30n205aglt.jpg)

附 https://golang.org/doc/faq#convert_slice_of_interface 对这个问题的解释：

![image-20201209203306491](https://tva1.sinaimg.cn/large/0081Kckwgy1glhvrg2vyfj31dy0f6acs.jpg)



<hr />
