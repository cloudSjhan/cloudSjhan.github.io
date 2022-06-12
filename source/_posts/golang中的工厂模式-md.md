---
title: golang中的工厂模式
tags: [golang,设计模式]
copyright: true
date: 2018-08-27 18:53:24
permalink:
categories: golang
description:
image:
---
<p class="description"></p>

<img src="https://" alt="" style="width:100%" />

<!-- more -->

* 研究go的设计模式，必须了解go的struct和interface，若不熟悉，先阅读以下内容
* [go语言的struct](http://blog.csdn.net/wangshubo1989/article/details/70040022)
* [go语言的interface](http://blog.csdn.net/wangshubo1989/article/details/70053086)

```go
* 简单工厂模式

package main

import (
    "fmt"
)

type Operater interface {
    Operate(int, int) int
}

type AddOperate struct {
}

func (this *AddOperate) Operate(rhs int, lhs int) int {
    return rhs + lhs
}

type MultipleOperate struct {
}

func (this *MultipleOperate) Operate(rhs int, lhs int) int {
    return rhs * lhs
}

type OperateFactory struct {
}

func NewOperateFactory() *OperateFactory {
    return &OperateFactory{}
}

func (this *OperateFactory) CreateOperate(operatename string) Operater {
    switch operatename {
    case "+":
        return &AddOperate{}
    case "*":
        return &MultipleOperate{}
    default:
        panic("无效运算符号")
        return nil
    }
}

func main() {
    Operator := NewOperateFactory().CreateOperate("+")
    fmt.Printf("add result is %d\n", Operator.Operate(1, 2))
}
```

```go
* 工厂方法
package main

import (
    "fmt"
)

type Operation struct {
    a float64
    b float64
}

type OperationI interface {
    GetResult() float64
    SetA(float64)
    SetB(float64)
}

func (op *Operation) SetA(a float64) {
    op.a = a
}

func (op *Operation) SetB(b float64) {
    op.b = b
}

type AddOperation struct {
    Operation
}

func (this *AddOperation) GetResult() float64 {
    return this.a + this.b
}

type SubOperation struct {
    Operation
}

func (this *SubOperation) GetResult() float64 {
    return this.a - this.b
}

type MulOperation struct {
    Operation
}

func (this *MulOperation) GetResult() float64 {
    return this.a * this.b
}

type DivOperation struct {
    Operation
}

func (this *DivOperation) GetResult() float64 {
    return this.a / this.b
}

type IFactory interface {
    CreateOperation() Operation
}

type AddFactory struct {
}

func (this *AddFactory) CreateOperation() OperationI {
    return &(AddOperation{})
}

type SubFactory struct {
}

func (this *SubFactory) CreateOperation() OperationI {
    return &(SubOperation{})
}

type MulFactory struct {
}

func (this *MulFactory) CreateOperation() OperationI {
    return &(MulOperation{})
}

type DivFactory struct {
}

func (this *DivFactory) CreateOperation() OperationI {
    return &(DivOperation{})
}

func main() {
    fac := &(AddFactory{})
    oper := fac.CreateOperation()
    oper.SetA(1)
    oper.SetB(2)
    fmt.Println(oper.GetResult())
}
```



```go
* 抽象工厂方法
package main

import "fmt"

type GirlFriend struct {
    nationality string
    eyesColor   string
    language    string
}

type AbstractFactory interface {
    CreateMyLove() GirlFriend
}

type IndianGirlFriendFactory struct {
}

type KoreanGirlFriendFactory struct {
}

func (a IndianGirlFriendFactory) CreateMyLove() GirlFriend {
    return GirlFriend{"Indian", "Black", "Hindi"}
}

func (a KoreanGirlFriendFactory) CreateMyLove() GirlFriend {
    return GirlFriend{"Korean", "Brown", "Korean"}
}

func getGirlFriend(typeGf string) GirlFriend {

    var gffact AbstractFactory
    switch typeGf {
    case "Indian":
        gffact = IndianGirlFriendFactory{}
        return gffact.CreateMyLove()
    case "Korean":
        gffact = KoreanGirlFriendFactory{}
        return gffact.CreateMyLove()
    }
    return GirlFriend{}
}

func main() {

    a := getGirlFriend("Indian")

    fmt.Println(a.eyesColor)
}
```



<hr />
