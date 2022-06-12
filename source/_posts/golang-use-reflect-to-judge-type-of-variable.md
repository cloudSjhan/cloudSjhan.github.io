---
title: golang use reflect to judge type of variable
tags: [golang]
copyright: true
date: 2018-12-25 19:38:43
permalink:
categories: go
description: golang use reflect to judge type of variable
image: https://static001.geekbang.org/resource/image/55/f9/55dcbbdf96451508691b2aef077e12f9.jpg
---
<p class="description"></p>

<img src="https://" alt="" style="width:100%" />

<!-- more -->

- 众所周知，golang中可以使用空接口即interface{}代表任何类型的数据，那么在使用的时候，我们有时需要获取返回值的具体类型
- 场景：beego框架中的orm.Params类型，实际上是map[string]interface{},在使用[values接口](https://beego.me/docs/mvc/model/query.md)的时候，需要从返回Map中获取数据，需要这样获取：`Id:m["Id"].(string)`,这时m["Id"]实际上是String类型，我们可以用reflect模块来获取实际的类型

```go
reflect.TypeOf(m["Id"])
//返回为String
```



```
package main

import (
    "fmt"
    "reflect"
)

func main() {
    var x int32 = 20
    fmt.Println("type:", reflect.TypeOf(x))
}
```



<hr />
