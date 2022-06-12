---
title: 从冒泡排序优化到Go基准测试
tags: [技术周刊, go]
copyright: true
date: 2018-12-15 23:12:08
permalink:
categories: 技术周刊
description: 从冒泡排序看Go的基准测试及其使用
image: https://static001.geekbang.org/resource/image/4d/ec/4d642c8e43980c1e35afc78440436cec.jpg
---
<p class="description"></p>

<img src="https://" alt="" style="width:100%" />

<!-- more -->

### Go的测试

#### 单元测试

- 测试文件命名：文件命名使用 xx_test.go 保存在项目目录里即可，也可以新建个test目录，TestAll；
- 测试函数命名：单元测试函数名Test开头，接收一个指针型参数（*testing.T）；
- 运行测试程序：go run test -v -run="函数名"，其中-v意思是输出详细的测试信息；

#### 基准测试

- 测试文件命名：测试文件命名：文件命名使用 xx_test.go 保存在项目目录里即可，也可以新建个test目录，TestAll；
- 测试函数命名：基准测试以Benchmark开头，接收一个指针型参数（*testing.B）；
- 运行测试程序：go test -v -bench="函数名"；
- 还有一个参数是 -benchmem, -benchmem 表示分配内存的次数和字节数，-benchtime="3s" 表示持续3秒

### 冒泡排序及其简单优化

#### 冒泡排序

前几天看一个公众号，说是有个人去美团面试，被要求手写冒泡排序算法，这人心想这还不简单的，分分钟写下了下面的代码，可以说是非常标准的冒泡排序了

```go
func BubbleSort(array []int)  {
	i := 0
	for i = 0; i<len(array)-1;i++{
		for j :=0;j<len(array)-i-1;j++{
			if array[j]>array[j+1]{
				swap(j, j+1)
				
			}
		}
		
	}
}
```

但是随之，面试官说让他优化一下这个算法，问他有没有可优化的地方，这人就懵逼了。其实冒泡排序可优化的地方很多，最简单的一种就是加一个标志位，检查是否已经排序完毕，排序已经完成的话就没有必要再比较下去，浪费时间。上面的代码添加几行，那人就很可能拿到offer了。

```go
func BubbleSort(array []int)  {
	i := 0
	for i = 0; i<len(array)-1;i++{
		exchanged := false
		for j :=0;j<len(array)-i-1;j++{
			if array[j]>array[j+1]{
				swap(j, j+1)
				exchanged = true
			}
		}
		if exchanged == false {
			return
		}
	}
}
```

那么，如何更直观的看出，优化过的代码确实有效呢，我们就可以使用go自带的基准测试，测试一下两段代码的性能，优化与否就一目了然。

### 基准测试冒泡排序

- 首先我们创建一个存放测试文件的文件夹

  ```go
  mkdir testAll
  ```

- 然后创建基准测试文件

```go
touch benchmark_test.go
```

- 编写基准测试代码

```
var sortArray = []int{41, 24, 76, 11, 45, 64, 21, 69, 19, 36}
func BenchmarkBubbleSort(b *testing.B)  {

	for i:=0;i<b.N;i++ {
	BubbleSort(sortArray)
	}
}
```



#### 测试基本冒泡排序

- 我们首先测试最原始的冒泡排序

- 在testAll文件夹下执行

  ```
  go test -v -bench="BubbleSort"
  ```

  ![](https://ws4.sinaimg.cn/large/006tNbRwly1fy7v2wibo3j313o0bgta3.jpg)

  基准测试结果如图所示，解释一下几个关键参数，

  81.2ns/op 表示每次操作耗时81.2纳秒，1.725s表示程序运行的时间。

  ### 测试优化的冒泡排序

  - 执行

  ```
  go test -v -bench="BubbleSort"
  ```

  ![](https://ws3.sinaimg.cn/large/006tNbRwly1fy7v8m1tz8j31400bcmyk.jpg)

   



  可以看到优化过的代码，执行时间为1.2s,而且每次操作的耗时为23.0纳秒，足足降低了60%多。可见一个很简单的优化就可以使代码提升这么多，在以后的工作中，对于代码的设计与优化还是要重视。


<hr />





