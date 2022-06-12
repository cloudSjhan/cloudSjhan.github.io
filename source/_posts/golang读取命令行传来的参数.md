---
title: golang读取命令行传来的参数
tags: [golang, go]
copyright: true
date: 2018-11-06 14:59:04
permalink:
categories: golang
description: golang使用命令行参数
image: https://static001.geekbang.org/resource/image/5e/ff/5e49573ffc2a18e700ed73ba3b4dc4ff.jpg
---
<p class="description"></p>

<img src="https://" alt="" style="width:100%" />

<!-- more -->

# Golang-使用命令行参数

Golang有两个标准包中都有获得命令行参数的方法：

```
[*]os/Args可以简单地获得一个类似Ｃ语言中的argv结构
[*]flag则提供了一个更为复杂的标志与值的方法
```

os.Argsos.Args返回一个字符串数组[] string.

使用方法很简单：package main

```
import (
    "fmt"
    "os"
)

func main() {
    fmt.Println(os.Args)
}
```

使用命令：go run test.go arg1 arg2

可见返回了一个三个元素的数组，第０个元素是程序的名字包括路径，os.Args就第一个参数，os.Args就是第二个参数。

------

flag包flag包提供的功能非常复杂。

它将命令行参数分为非标志类参数(nonflag arguments)和Flags，标志参数是这样的-flagname=x，比如说-baudrate=1200。

非标志类参数为arg1 arg2。

flag参数处理流程由于标志类参数是参数的一部分，但又特殊，为了将标志类参数区别处理

flag包有两类方法，一类是flag处理方法，另一类是正常的参数处理方法。

正常的参数处理方法正常参数处理方法与os.Args差不多，这里是一个方法，flag.Args()，返回也是[]string.

```
package main
import (
    "fmt"
/*    "os/exec"
    "bytes"*/
    "flag"
)

func main() {
    flag.Parse()
    fmt.Println(flag.Args())
} 

go run test.go arg1 arg2
```

如果有标志类参数呢？

```
go run test.go arg1 arg2　-baudrate=1200
```

这里充分证明了标志类参数也是参数。

标志类参数Parse前定义如果使用标志类参数，要提前定义,定义之后再调用Parse才能解析出来：

```
package main
import (
    "fmt"
    "flag"
)

func main() {
    baudrate:=flag.Int("baudrate",1200, "help message for flagname")
    databits:=flag.Int("databits",10,"number of data bits")
    flag.Parse()
    fmt.Println(*baudrate)
    fmt.Println(*databits)
    fmt.Println(flag.Args())
}

go run test.go -baudrate=9600 -databits=8 arg1 arg2
```

标志类参数必须在Parse之定义，否则会出错：

```
package main
import (
    "fmt"
    "flag"
)

func main() {
    flag.Parse()
    baudrate:=flag.Int("baudrate",1200, "help message for flagname")
    databits:=flag.Int("databits",10,"number of data bits")
    fmt.Println(*baudrate)
    fmt.Println(*databits)
    fmt.Println(flag.Args())
}

go run test.go -baudrate=9600 -databits=8 arg1 arg2

flag provided but not defined: -baudrate

Usage of /tmp/go-build944578075/command-line-arguments/_obj/a.out:
exit status 2
```

flag.Int返回的是地址

需要注意的是这里flag.Int返回的值为一个地址，你可以随时到这个地址里去取值

但在Parse之前取值，取到的是默认值，Parse之后去随值，取到的才是真正的值：

```
package main
import (
    "fmt"
    "flag"
)

func main() {
    baudrate:=flag.Int("baudrate",1200, "help message for flagname")
    databits:=flag.Int("databits",10,"number of data bits")

    fmt.Println(*baudrate)
    fmt.Println(*databits)
    flag.Parse()
    fmt.Println(flag.Args())
}

go run test.go -baudrate=9600 -databits=8 arg1 arg2
```

标志类参数顺序

标志类参数之间的前后顺序可以改变，但是似乎标志类参数非要放到非标志类参数之前才能正确解析。

```
package main
import (
    "fmt"
    "flag"
)

func main() {
    databits:=flag.Int("databits",10,"number of data bits")
    baudrate:=flag.Int("baudrate",1200, "help message for flagname")
    flag.Parse()
    fmt.Println(*baudrate)
    fmt.Println(*databits)
    fmt.Println(flag.Args())
}

go run test.go -baudrate=9600 -databits=8 arg1 arg2
```

上面的命令正确解析了，调换了baudrate和databits的顺序

```
go run test.go arg1 -baudrate=9600 -databits=8  arg2
```

上前这里没能正确解析，可以baudrate和databits得到的还是默认值，而非标志类参数获取到了所有的参数。

--help

flag.Int的最后一个参数是help信息：

```
go run test.go --help

Usage of /tmp/go-build327358548/command-line-arguments/_obj/a.out:
  -baudrate=1200: help message for flagname
  -databits=10: number of data bits
exit status 2
```

flag.String传入的参数显然不能都是数字，实际go语言提供的类型都支持，与flag.Int类似，所有其他函数都有：

```
flag.String flag.Uint flag.Float64....
```

flag.IntVarflag.Int返回的是指针，用起来可以有点不太好，flag.IntVar可能用起来更好的些：

```
var baudrate int
flag.IntVar(&baudrate,"baudrate",1200,"baudrate of serial port")
flag.Parse()
fmt.Println(baudrate)
```

当前你一样可以用flag.UintVar flag.Float64Var flag.StringVar

参数个数参数个数也分为标志类参数的非标志类参数，两个方法为NArg和NFlag,

```go
package main
import (
    "fmt"
    "flag"
)

func main() {
    databits:=flag.Int("databits",10,"number of data bits")
    baudrate:=flag.Int("baudrate",1200, "help message for flagname")
    flag.Parse()
    fmt.Println(*baudrate)
    fmt.Println(*databits)
    fmt.Println(flag.Args())
    fmt.Println(flag.NArg())
    fmt.Println(flag.NFlag())
}

go run test.go -baudrate=9600 -databits=8 arg1 arg2
```

以上代码的执行的过程以及执行结果是：

![](https://ws3.sinaimg.cn/large/006tNbRwgy1fwycwsdqmrj31ks09iq3t.jpg)

从上到下打印出的参数含义分别是：

1111：指定的标志类参数baudrate，默认值是1200，可随意更改；

1011： 指定的标志类参数databits，默认值是10，可随意更改；

[la, la]:非标志类参数为arg1 arg2；

2：非标志类参数的数量

2：标志类参数的数量

  

​                                                          The End!



<hr />
