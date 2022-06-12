---

title: 技术周刊之golang中修改struct的slice的值
tags: [golang,技术周刊]
copyright: true
date: 2018-10-27 10:30:10
permalink:
categories: 技术周刊
description: 如何优雅地修改go中struct的slice的值
image: https://static001.geekbang.org/resource/image/ed/02/edb8e8c857c26b85273eb83d58b68d02.jpg
---
<p class="description"></p>

<img src="https://" alt="" style="width:100%" />

<!-- more -->

### 前言

- 前段时间写go的时候遇到一个问题，需要修改由struct构成的slice中struct的某个字段值，类似于下面的需求：

```go
type Docker struct {
	Ip  string
	ID string
}


docker1 := Docker{
		Ip:  "222",
		ID: "aaa",
	}

	docker2 := Docker{
		Ip:  "111",
		ID: "bbb",
	}
	
	var tmpDocker []Docker
	tmpDocker = append(tmpDocker, docker1)
	tmpDocker = append(tmpDocker, docker2)
	现在需要修改tmpDocker中，Ip这个字段的值, 你可以先自己尝试修改一下，然后再往下看
```



由这个问题我查阅很多资料，我们先从语言中经典的传值、传引用说起来

- 对于一门语言，我们关心传递参数的过程中，是传值还是传引用，其实对于传值和传引用是一个比较古老的问题，在大学入门的时候，你可能就接触过这样的C语言代码：

```c
//以下哪个函数能实现交换两个数？
#include<iostream>
using namespace std;
 
void swap1(int p,int q)
{
    int temp;
    temp=p;
    p=q;
    q=temp;
}
 
void swap2(int *p,int *q)
{
    int *temp;
    *temp=*p;
    *p=*q;
    *q=*temp;
}

```

其实对于C语言来说，并没有传引用的概念，看似传引用的操控，实际上传的是指针的地址，也算是一种传值，先看一下传值，传引用，传指针的概念：

- 传值：可能很多人都听说，传值无非就是实参拷贝传递给形参。这句话没有错，但是理解起来还是有点抽象。一句话，传值就是把实参赋值给形参，赋值完毕后实参就和形参没有任何联系，对形参的修改就不会影响到实参。
- 传地址：为什么说传地址也是一种传值呢？因为传地址是把实参地址的拷贝传递给形参。还是一句话，传地址就是把实参的地址复制给形参。复制完毕后实参的地址和形参的地址没有任何联系，对实参形参地址的修改不会影响到实参, 但是对形参地址所指向对象的修改却直接反应在实参中，因为形参指向的对象就是形参的对象。
- 传引用：传引用本质没有任何实参的拷贝，一句话，就是让另外一个变量也执行该实参。就是两个变量指向同一个对象。这是对形参的修改，必然反映到实参上。

那么对于go语言来说，是没有引用传递的，go作为云计算时代的C语言，采用的都是值传递，即使是指针，也是将指针的地址即指针的指针，拷贝一份传递，可以参考这篇博文的讲解：Go[语言参数传递是传值还是传引用](http://www.flysnow.org/2018/02/24/golang-function-parameters-passed-by-value.html)

### 回到正题

- 了解基本的知识背景之后，让我们回到文章开头的代码，即要修改slice中struct某字段的值。

  ```
  type Docker struct {
  	Ip  string
  	ID string
  }
  
  docker1 := Docker{
  		Ip:  "222",
  		ID: "aaa",
  	}
  
  	docker2 := Docker{
  		Ip:  "111",
  		ID: "bbb",
  	}
  	
  	var tmpDocker []Docker
  	tmpDocker = append(tmpDocker, docker1)
  	tmpDocker = append(tmpDocker, docker2)
  ```

  先将我最初的代码实现贴出来：

  ```go
  
  	for _, dockerInfo := range tmpDocker {
  		dockerInfo.Ip = "192.168,.1.1"
  	}
  	fmt.Println(tmpDocker)
  ```

  让我们看一下运行结果：

  ![](https://ws2.sinaimg.cn/large/006tNbRwly1fwmm2z70f6j30d301ja9z.jpg)

  发现struct中Ip字段的值并没有改变，是为什么呢？

  原因就是：range的过程中产生了一个新的对象，即dockerInfo是temDocker中每个元素的一个副本，所以你改变的只是副本中Ip字段的值，并没有改变真实的。那么如何解决呢?

  这里我提出两种解决的方法，代码如下：

  ```go
  //方法1：赋给一个新的对象
  newTmpDocker := []Docker{}//新的对象
  for _, dockerInfo := range tmpDocker {
      dockerInfo.Ip = "192.168.1.1"
      newTmpDocker = append(newTmpDocker, dockerInfo)
  }
  fmt.Println(newTmpDocker)
  ```

  ![](https://ws3.sinaimg.cn/large/006tNbRwly1fwmnl32x44j30dj01imx3.jpg)

  可以看到最终输出的struct的slice中我们想要改变的字段已经修改成功。

  第二种方法是将副本修改后赋值

  ```go
  方法2：修改副本后，将副本赋值给原来的
  for i, dockerInfo := range tmpDocker{
      dockerInfo.Ip = "192.168.1.1"
      tmpDocker[i] = dockerInfo
  }
  fmt.Println(tmpDocker)
  
  
  ```

  运行结果：

  ![](https://ws3.sinaimg.cn/large/006tNbRwly1fwmnl32x44j30dj01imx3.jpg)

  可以看到同样修改成功。

### 总结，在go中，所有传参都是传值，都是一个副本，即一个拷贝，因为拷贝的内容有时候是非引用类型（int、string、struct等这些），这样就在函数中就无法修改原内容数据；有的是引用类型（指针、map、slice、chan等这些），这样就可以修改原内容数据。

是否可以修改原内容数据，和传值、传引用没有必然的关系。在C++中，传引用肯定是可以修改原内容数据的，在Go语言里，虽然只有传值，但是我们也可以修改原内容数据，因为参数是引用类型。

这里也要记住，引用类型和传引用是两个概念。

再记住，Go里只有传值（值传递）。

<hr />
