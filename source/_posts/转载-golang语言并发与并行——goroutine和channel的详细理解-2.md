---
title: (转载)golang语言并发与并行——goroutine和channel的详细理解(2)
tags: [golang]
copyright: true
date: 2018-12-07 11:48:11
permalink:
categories: golang
description: golang语言并发与并行——goroutine和channel的详细理解
image: https://static001.geekbang.org/resource/image/a6/d6/a60c994bf629abe46076f77db6ff24d6.jpg
---
<p class="description"></p>

<img src="https://" alt="" style="width:100%" />

<!-- more -->

[本文转载自](https://studygolang.com/articles/9533)，版权属于原作者。

# Go语言的并发和并行

不知道你有没有注意到一个现象，还是这段代码，如果我跑在两个goroutines里面的话:

```
var quit chan int = make(chan int)

func loop() {
    for i := 0; i < 10; i++ {
        fmt.Printf("%d ", i)
    }
    quit <- 0
}


func main() {
    // 开两个goroutine跑函数loop, loop函数负责打印10个数
    go loop()
    go loop()

    for i := 0; i < 2; i++ {
        <- quit
    }
}
```

我们观察下输出:

```
0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9
```

这是不是有什么问题？?

以前我们用线程去做类似任务的时候，系统的线程会抢占式地输出， 表现出来的是乱序地输出。而goroutine为什么是这样输出的呢？

## goroutine是在并行吗？

我们找个例子[测试](http://lib.csdn.net/base/softwaretest)下:

```
package main

import "fmt"
import "time"

var quit chan int

func foo(id int) {
    fmt.Println(id)
    time.Sleep(time.Second) // 停顿一秒
    quit <- 0 // 发消息：我执行完啦！
}


func main() {
    count := 1000
    quit = make(chan int, count) // 缓冲1000个数据

    for i := 0; i < count; i++ { //开1000个goroutine
        go foo(i)
    }

    for i :=0 ; i < count; i++ { // 等待所有完成消息发送完毕。
        <- quit
    }
}
```

让我们跑一下这个程序(之所以先编译再运行，是为了让程序跑的尽量快,测试结果更好):

```
go build test.go
time ./test
./test  0.01s user 0.01s system 1% cpu 1.016 total
```

我们看到，总计用时接近一秒。 貌似并行了！

我们需要首先考虑下什么是并发, 什么是并行

## 并行和并发

从概念上讲，并发和并行是不同的, 简单来说看这个图片(原图来自[这里](http://joearms.github.io/2013/04/05/concurrent-and-parallel-programming.html))

![img](http://hit9.qiniudn.com/con_and_par.jpg)

- 两个队列，一个Coffee机器，那是并发
- 两个队列，两个Coffee机器，那是并行

更多的资料： [并发不是并行](http://www.aqee.net/docs/Concurrency-is-not-Parallelism/), 当然Google上有更多关于并行和并发的区别。

那么回到一开始的疑问上，从上面的两个例子执行后的表现来看，多个goroutine跑loop函数会挨个goroutine去进行，而sleep则是一起执行的。

这是为什么？

默认地， [Go](http://lib.csdn.net/base/go)所有的goroutines只能在一个线程里跑 。

也就是说， 以上两个代码都不是并行的，但是都是是并发的。

如果当前goroutine不发生阻塞，它是不会让出CPU给其他goroutine的, 所以例子一中的输出会是一个一个goroutine进行的，而sleep函数则阻塞掉了 当前goroutine, 当前goroutine主动让其他goroutine执行, 所以形成了逻辑上的并行, 也就是并发。

## 真正的并行

为了达到真正的并行，我们需要告诉Go我们允许同时最多使用多个核。

回到起初的例子，我们设置最大开2个原生线程, 我们需要用到runtime包(runtime包是goroutine的调度器):

```
import (
    "fmt"
    "runtime"
)

var quit chan int = make(chan int)

func loop() {
    for i := 0; i < 100; i++ { //为了观察，跑多些
        fmt.Printf("%d ", i)
    }
    quit <- 0
}

func main() {
    runtime.GOMAXPROCS(2) // 最多使用2个核

    go loop()
    go loop()

    for i := 0; i < 2; i++ {
        <- quit
    }
}
```

这下会看到两个goroutine会抢占式地输出数据了。

我们还可以这样显式地让出CPU时间：

```
func loop() {
    for i := 0; i < 10; i++ {
        runtime.Gosched() // 显式地让出CPU时间给其他goroutine
        fmt.Printf("%d ", i)
    }
    quit <- 0
}


func main() {

    go loop()
    go loop()

    for i := 0; i < 2; i++ {
        <- quit
    }
}
```

观察下结果会看到这样有规律的输出:

```
0 0 1 1 2 2 3 3 4 4 5 5 6 6 7 7 8 8 9 9
```

其实，这种主动让出CPU时间的方式仍然是在单核里跑。但手工地切换goroutine导致了看上去的“并行”。

其实作为一个[Python](http://lib.csdn.net/base/python)程序员，goroutine让我更多地想到的是gevent的协程，而不是原生线程。

关于runtime包对goroutine的调度，在stackoverflow上有一个不错的答案:<http://stackoverflow.com/questions/13107958/what-exactly-does-runtime-gosched-do>

## 一个小问题

我在Segmentfault看到了这个问题: <http://segmentfault.com/q/1010000000207474>

题目说，如下的程序，按照理解应该打印下5次 `"world"`呀，可是为什么什么也没有打印

```
package main

import (
    "fmt"
)

func say(s string) {
    for i := 0; i < 5; i++ {
        fmt.Println(s)
    }
}

func main() {
    go say("world") //开一个新的Goroutines执行
    for {
    }
}
```

楼下的答案已经很棒了，这里Go仍然在使用单核，for死循环占据了单核CPU所有的资源，而main线和say两个goroutine都在一个线程里面， 所以say没有机会执行。解决方案还是两个：

- 允许Go使用多核(`runtime.GOMAXPROCS`)
- 手动显式调动(`runtime.Gosched`)

## runtime调度器

runtime调度器是个很神奇的东西，但是我真是但愿它不存在，我希望显式调度能更为自然些，多核处理默认开启。

关于runtime包几个函数:

- `Gosched` 让出cpu
- `NumCPU` 返回当前系统的CPU核数量
- `GOMAXPROCS` 设置最大的可同时使用的CPU核数
- `Goexit` 退出当前goroutine(但是defer语句会照常执行)

## 总结

我们从例子中可以看到，默认的, 所有goroutine会在一个原生线程里跑，也就是只使用了一个CPU核。

在同一个原生线程里，如果当前goroutine不发生阻塞，它是不会让出CPU时间给其他同线程的goroutines的，这是Go运行时对goroutine的调度，我们也可以使用runtime包来手工调度。

本文开头的两个例子都是限制在单核CPU里执行的，所有的goroutines跑在一个线程里面，分析如下:

- 对于代码例子一（loop函数的那个），每个goroutine没有发生堵塞(直到quit流入数据), 所以在quit之前每个goroutine不会主动让出CPU，也就发生了串行打印
- 对于代码例子二（time的那个），每个goroutine在sleep被调用的时候会阻塞，让出CPU, 所以例子二并发执行。

那么关于我们开启多核的时候呢？Go语言对goroutine的调度行为又是怎么样的？

我们可以在Golang官方网站的[这里](http://golang.org/doc/faq#goroutines) 找到一句话:

> When a coroutine blocks, such as by calling a blocking system call, the run-time automatically moves other coroutines on the same operating system thread to a different, runnable thread so they won't be blocked.

也就是说:

> 当一个goroutine发生阻塞，Go会自动地把与该goroutine处于同一系统线程的其他goroutines转移到另一个系统线程上去，以使这些goroutines不阻塞

## 开启多核的实验

仍然需要做一个实验，来测试下多核支持下goroutines的对原生线程的分配, 也验证下我们所得到的结论“goroutine不阻塞不放开CPU”。

实验代码如下:

```
package main

import (
    "fmt"
    "runtime"
)

var quit chan int = make(chan int)

func loop(id int) { // id: 该goroutine的标号
    for i := 0; i < 10; i++ { //打印10次该goroutine的标号
        fmt.Printf("%d ", id)
    }
    quit <- 0
}

func main() {
    runtime.GOMAXPROCS(2) // 最多同时使用2个核

    for i := 0; i < 3; i++ { //开三个goroutine
        go loop(i)
    }

    for i := 0; i < 3; i++ {
        <- quit
    }
}
```

多跑几次会看到类似这些输出(不同机器环境不一样):

```
0 0 0 0 0 1 1 0 0 1 0 0 1 0 1 2 1 2 1 2 1 2 1 2 1 2 2 2 2 2
0 0 0 0 0 0 0 0 0 0 1 1 1 1 1 1 1 1 1 1 2 2 2 2 2 2 2 2 2 2
0 0 0 0 0 0 0 1 1 1 1 1 0 1 0 1 0 1 2 1 2 1 2 2 2 2 2 2 2 2
0 0 0 0 0 0 0 1 1 1 1 1 1 1 1 1 1 0 2 0 2 0 2 2 2 2 2 2 2 2
0 0 0 0 0 0 0 1 0 0 1 0 1 2 1 2 1 2 1 2 1 2 1 2 1 2 1 2 2 2
```

执行它我们会发现以下现象:

- 有时会发生抢占式输出(说明Go开了不止一个原生线程，达到了真正的并行)
- 有时会顺序输出, 打印完0再打印1, 再打印2(说明Go开一个原生线程，单线程上的goroutine不阻塞不松开CPU)

那么，我们还会观察到一个现象，无论是抢占地输出还是顺序的输出，都会有那么两个数字表现出这样的现象:

- 一个数字的所有输出都会在另一个数字的所有输出之前

原因是， 3个goroutine分配到至多2个线程上，就会至少两个goroutine分配到同一个线程里，单线程里的goroutine 不阻塞不放开CPU, 也就发生了顺序输出。

<hr />
