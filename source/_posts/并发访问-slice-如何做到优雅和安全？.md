---
title: 并发访问 slice 如何做到优雅和安全？
tags: [go]
copyright: true
date: 2020-04-22 11:43:15
permalink:
categories: go
description: 并发访问 slice 如何做到优雅和安全？
image:
---
<p class="description"></p>

<img src="https://" alt="" style="width:100%" />

<!-- more -->

### 抛出问题

由于 slice/map 是引用类型，golang函数是传值调用，所用参数副本依然是原来的 slice， 并发访问同一个资源会导致竟态条件。

看下面这段代码：

```go
package main

import (
    "fmt"
    "sync"
)

func main() {
    var (
        slc = []int{}
        n   = 10000
        wg  sync.WaitGroup
    )

    wg.Add(n)
    for i := 0; i < n; i++ {
        go func() {
            slc = append(slc, i)
            wg.Done()
        }()
    }
    wg.Wait()

    fmt.Println("len:", len(slc))
    fmt.Println("done")
}

// Output:
len: 8586
done
```

真实的输出并没有达到我们的预期，len(slice) < n。 问题出在哪？我们都知道slice是对数组一个连续片段的引用，当slice长度增加的时候，可能底层的数组会被换掉。当出在换底层数组之前，切片同时被多个goroutine拿到，并执行append操作。那么很多goroutine的append结果会被覆盖，导致n个gouroutine append后，长度小于n。

那么如何解决这个问题呢？
map 在 go 1.9 以后官方就给出了 sync.map 的解决方案，但是如果要并发访问 slice 就要自己好好设计一下了。下面提供两种方式，帮助你解决这个问题。

### 方案 1: 加锁 🔐

```go
func main() {
	slc := make([]int, 0, 1000)
	var wg sync.WaitGroup
	var lock sync.Mutex

	for i := 0; i < 1000; i++ {
		wg.Add(1)
		go func(a int) {
			defer wg.Done()
      // 加🔐
			lock.Lock()
			defer lock.Unlock()
			slc = append(slc, a)
		}(i)
		wg.Wait()

	}

	fmt.Println(len(slc))
}
```

> 优点是比较简单，适合对性能要求不高的场景。



### 方案 2： 使用 channel 串行化操作

```go
type ServiceData struct {
	ch   chan int // 用来 同步的channel
	data []int    // 存储数据的slice
}

func (s *ServiceData) Schedule() {
	// 从 channel 接收数据
	for i := range s.ch {
		s.data = append(s.data, i)
	}
}

func (s *ServiceData) Close() {
	// 最后关闭 channel
	close(s.ch)
}

func (s *ServiceData) AddData(v int) {
	s.ch <- v // 发送数据到 channel
}

func NewScheduleJob(size int, done func()) *ServiceData {
	s := &ServiceData{
		ch:   make(chan int, size),
		data: make([]int, 0),
	}

	go func() {
		// 并发地 append 数据到 slice
		s.Schedule()
		done()
	}()

	return s
}

func main() {
	var (
		wg sync.WaitGroup
		n  = 1000
	)
	c := make(chan struct{})

	// new 了这个 job 后，该 job 就开始准备从 channel 接收数据了
	s := NewScheduleJob(n, func() { c <- struct{}{} })

	wg.Add(n)
	for i := 0; i < n; i++ {
		go func(v int) {
			defer wg.Done()
			s.AddData(v)

		}(i)
	}

	wg.Wait()
	s.Close()
	<-c

	fmt.Println(len(s.data))
}
```

> 实现相对复杂，优点是性能很好，利用了channel的优势

以上代码都有比较详细的注释，就不展开讲了。


![](https://user-gold-cdn.xitu.io/2020/4/15/1717c5c5de0af19a?w=227&h=227&f=jpeg&s=20015)

<hr />
