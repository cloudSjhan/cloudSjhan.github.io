---
title: go语言坑之for range
tags: [go]
copyright: true
date: 2019-01-31 09:59:59
permalink:
categories: go
description: go语言坑之for range
image: https://avatars1.githubusercontent.com/u/4314092?s=200&v=4
---
<p class="description"></p>

<img src="https://" alt="" style="width:100%" />

<!-- more -->

# go语言坑之for range
> 这篇文章简单清晰地解释了之前遍历struct切片时遇到的一些怪异现象，这个问题当时也写了一篇文章来讨论，[文章地址](https://cloudsjhan.github.io/2018/10/27/%E6%8A%80%E6%9C%AF%E5%91%A8%E5%88%8A%E4%B9%8Bgolang%E4%B8%AD%E4%BF%AE%E6%94%B9struct%E7%9A%84slice%E7%9A%84%E5%80%BC/)，但是没有切中要点。昨天无意中看到这篇博客，豁然开朗。
---

go只提供了一种循环方式，即for循环，在使用时可以像c那样使用，也可以通过for range方式遍历容器类型如数组、切片和映射。但是在使用for range时，如果使用不当，就会出现一些问题，导致程序运行行为不如预期。比如，下面的示例程序将遍历一个切片，并将切片的值当成映射的键和值存入，切片类型是一个int型，映射的类型是键为int型，值为*int，即值是一个地址。

```
package main

import "fmt"

func main() {
    slice := []int{0, 1, 2, 3}
    myMap := make(map[int]*int)

    for index, value := range slice {
        myMap[index] = &value
    }
    fmt.Println("=====new map=====")
    prtMap(myMap)
}

func prtMap(myMap map[int]*int) {
    for key, value := range myMap {
        fmt.Printf("map[%v]=%v\n", key, *value)
    }
}
```

运行程序输出如下：

```
=====new map=====
map[3]=3
map[0]=3
map[1]=3
map[2]=3
```

由输出可以知道，不是我们预期的输出，正确输出应该如下：

```
=====new map=====
map[0]=0
map[1]=1
map[2]=2
map[3]=3
（无序输出，但是值是0,1,2,3）
```

但是由输出可以知道，映射的值都相同且都是3。其实可以猜测映射的值都是同一个地址，遍历到切片的最后一个元素3时，将3写入了该地址，所以导致映射所有值都相同。其实真实原因也是如此，因为for range创建了每个元素的副本，而不是直接返回每个元素的引用，如果使用该值变量的地址作为指向每个元素的指针，就会导致错误，在迭代时，返回的变量是一个迭代过程中根据切片依次赋值的新变量，所以值的地址总是相同的，导致结果不如预期。

修正后程序如下：

```
package main

import "fmt"

func main() {
    slice := []int{0, 1, 2, 3}
    myMap := make(map[int]*int)

    for index, value := range slice {
        num := value
        myMap[index] = &num
    }
    fmt.Println("=====new map=====")
    prtMap(myMap)
}

func prtMap(myMap map[int]*int) {
    for key, value := range myMap {
        fmt.Printf("map[%v]=%v\n", key, *value)
    }
}
```

运行程序输出如下：

```shell
=====new map=====
map[2]=2
map[3]=3
map[0]=0
map[1]=1
```

---

引用声明：该博客来自于[这里](https://studygolang.com/articles/9701).

<hr />
