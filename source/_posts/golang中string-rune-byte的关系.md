---
title: 'golang中string,rune,byte的关系'
tags: [golang]
copyright: true
date: 2018-10-25 09:55:40
permalink:
categories: golang
description: 浅析golang中String，rune, byte的关系
image: https://ws4.sinaimg.cn/large/006tNbRwly1fwk9bb6yy8j31kw10cb2c.jpg
---
<p class="description"></p>

<img src="https://" alt="" style="width:100%" />

<!-- more -->

1. golang中String的底层是使用byte[]数组存储的，不可改变

```go
str := "Golang 测试"
fmt.Println(len(str))
```

这段代码按道理应该输出6+1+2.

实际运行之后输出却是13， 原因是中文字符在utf-8编码的系统中是3个字节存储的，在Unicode中是2个字节存储的，go的默认编码格式是utf-8，so。。

这时候，我们使用下标访问字符串中的中文字符是不行的，想要使用下标访问，就需要rune出马。

2. 在官方文档中，rune的定义是：

```
 rune is an alias for int32 and is equivalent to int32 in all ways. It is
 used, by convention, to distinguish character values from integer values.

int32的别名，几乎在所有方面等同于int32
它用来区分字符值和整数值
type rune int32

```

那么我们想要得到预期字符串的长度，就要使用rune切片来实现。

```go
fmt.Println("rune:", len([]rune(str)))
```

就会输出预期的rune: 9.

这时我们也可以按照下标去访问str中的字符了。即[7]rune(str) = "测"。

3.总结

string的底层是byte，byte与rune的不同之处是：

byte 等同于int8，常用来处理ascii字符
rune 等同于int32,常用来处理unicode或utf-8字符

或者可以这样说：

rune 能操作任何字符
byte 不支持中文的操作



​       									END

<hr />
