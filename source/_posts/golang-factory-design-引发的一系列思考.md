---
title: golang factory design 引发的一系列思考
tags: [golang design pattern go-interface]
copyright: true
date: 2018-08-29 10:21:56
permalink:
categories: 技术周刊
description: 工作需要，看了一下golang如何实现工厂模式，遇到一些难以理解的知识点，查资料，写demo验证后，记录下来以供参考
image:
---
<p class="description"></p>

<!-- more -->

- 写在前面，突然萌生一个念头，做一个技术周刊系列，将每周工作或者生活当中遇到的比较有趣的问题记录下来，一来时总结一下，二来是为了以后退役了，可以回顾自己的技术生涯。
- 没有什么意外的话，我会每周六晚更新。
- 最近在整合三家公有云（AWS，ali, ucloud）的接口，考虑到代码复用的问题，于是开始考虑使用一种设计模式，这种场景下，最合适的便是工厂模式，将三家厂商的公有接口放入工厂方法中，然后对每一家new一个实例即可，以后再有新的厂商加入，改动的代码也不会太多。但是设计模式这种东西天然适合于java，对于golang这种比较新的语言来说，实现起来相对没有那么容易，对于刚接触golang的我来说，对一些golang的特性上并不是很熟悉，所以在此期间遇到一些不解的问题，写出来分享一下。

### 首先，什么是工厂模式

* 简单工厂模式就是通过传递不同的参数，生成不同的实例，工厂方法为每一个product提供一个工程类，通过不同的工厂创建不同的实例。

### 典型工厂模式的实现方式（即典型oop实现方式）

* ```c++
  class ProviderModel{
      provider string
          func factory(providerName string, test string){
          if providerName == "AWS" {
              return new AWS(test)
          }
          if providerName == "Ali"{
              return new Ali(test)
          }
      }
  }
  class AWS extends ProviderModel {
      func construct(test string){
          this.test = test
      }
      func doRequest(){}
  }
  awsmodel := ProviderModel::factory("AWS")
  awsmodel.doRequest()
  
  alimodel := ProviderModel ::factory("Ali")  
  alimodel.doRequest()
  ```

### golang实现工厂模式存在的问题

* golang的特性中并没有像java一样的继承和重载，所以我们要利用golang存在的特性，透过工厂模式的表面透析其本质。

* 我们看一下工厂模式就知道，所谓工厂其实就是定义了一些需要去实现的方法，golang的interface正是可以做到。于是先到Google上搜了一段golang实现的工厂模式的代码。

  ```go
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

  代码看起来没什么问题，后来又看到一种实现方式，[来自这篇博客](https://www.jianshu.com/p/9de2cd9bf8f0)，代码如下：

  ```go
  type site interface {
      fetch()
  }
  
  type siteModel struct {
      URL string
  }
  type site1 struct {
      siteModel
  }
  
  func (s site1) fetch() {
      fmt.Println("site1 fetch data")
  }
  
  func factory(s string) site {
      if s == "site" {
          return site1{
              siteModel{URL: "http://www.xxxx.com"},
          }
      }
      return nil
  }
  
  func main() {
      s := factory("site")
      s.fetch()
  }
  
  ```

  代码初看上去跟第一个实现没什么不一样，但是当我详细阅读代码时，下面的这句代码着实把我弄晕了

  ```go
  func factory(s string) site {
      if s == "site" {
          return site1{
              siteModel{URL: "http://www.xxxx.com"},
          }
      }
      return nil
  }
  
  ```

  factory函数的返回值定义明明是一个interface, 但是在return的时候，却返回一个struct，查阅很多资料后，[这篇博客](http://legendtkl.com/2017/06/12/understanding-golang-interface/)帮了我的大忙，其中对interface的解释有这么一句话：**在 Golang 中，interface 是一组 method 的集合，是 duck-type programming 的一种体现。不关心属性（数据），只关心行为（方法）。具体使用中你可以自定义自己的 struct，并提供特定的 interface 里面的 method 就可以把它当成 interface 来使用。**之后又详细看了几遍这篇博文，犹如醍醐灌顶，对golanginterface的理解更深了一层。读完这篇后再去实现工厂模式，或者再去写golang的代码，对interface的使用就会更自如一些。

### 总结

* 本期技术周刊主要由golang工厂模式的讨论引起，之后又涉及到golang interface特性的讨论，对以后使用golang编写更加复杂的代码很有帮助。

* 本期结束，欲知后事如何，且看下周分解。

  ![](https://timgsa.baidu.com/timg?image&quality=80&size=b9999_10000&sec=1535796309427&di=a9db53cf71b492f4dd06a57b5ec65229&imgtype=jpg&src=http%3A%2F%2Fimg4.imgtn.bdimg.com%2Fit%2Fu%3D2705270329%2C1518266531%26fm%3D214%26gp%3D0.jpg)

<hr />
