---
title: github&Daocloud演讲稿整理与总结
tags: [golang,生活]
copyright: true
date: 2019-11-23 22:27:40
permalink:
categories: golang
description: github&Daocloud演讲总结
image: https://tva1.sinaimg.cn/large/006y8mN6gy1g98cmagdpsj31400u04qp.jpg
---
<p class="description"></p>
<img src="https://" alt="" style="width:100%" />

<!-- more -->

大家下午好，今天给大家带来的分享主题是 golang 的内存分析以及优化，主要分为两部分内容，一是 以开源项目crawlab的内存分析与优化为例，讲解golang内存调优的一些工具和分析的过程；

第二部分是介绍平时开发中容易忽略的和容易带来性能问题的点，以及如何去优化。

当时这个话题的灵感来自一个开源项目 crawlab, crawlab是一个分布式任务调度平台，这里不展开详细解释了，大家有兴趣可以到GitHub主页看一下，https://github.com/crawlab-team/crawlab



![crawlab_pic](https://tva1.sinaimg.cn/large/006y8mN6gy1g98bvonxkzj305k05kmx1.jpg)

当时是有用户报出crawlab启动一段时间后，主节点机器会出现内存占用过高的问题，一台4G内存的服务器在运行crawlab后竟然能占用3G以的内存，然后整个master就会down掉，于是我开始对crawlab的后端服务进行性能的分析，下面将带大家回顾这一过程。

首先介绍，Golang的性能分析主要分为两种方式，![截屏2019-11-2322.14.47](https://tva1.sinaimg.cn/large/006y8mN6gy1g98ckayzwxj323f0u0wui.jpg)

一种是通过文件方式输出profile，特点是灵活性高，适用于特定代码片段的分析，我们通过调用/runtime包中的/pprof的API生成相应的pprof文件，然后使用 pprof就可以进入类似GDB的窗口。在这个窗口中你就可以用top,list function等命令去查看cpu,memory的状况以及goroutine运行情况。

第二种方式就是通过HTTP输出profile，

![截屏2019-11-2322.17.33](https://tva1.sinaimg.cn/large/006y8mN6gy1g98ckr5ve9j31cz0u07h2.jpg)

这种方式就适合于你还不清楚到底哪一段代码出现问题，而且你的应用要持续运行的情况。在分析crawlab的时候，采用的是这种方式。首先，我们在crawlab项目中嵌入如下几行代码，将crawlab后端服务启动后，浏览器中输入http://ip:8899/debug/pprof/就可以看到一个汇总分析页面，显示如下信息，点击heap，在汇总分析页面的最上方可以看到如下图所示，



![heap_server](https://tva1.sinaimg.cn/large/006y8mN6gy1g98cjvmammj311k06w0t6.jpg)

红色箭头所指的就是当前已经使用的堆内存是25M,！在我只上传一个爬虫文件，而且这个文件还不是特别打大的情况下，一个后端服务所用的内存使用竟然能达到25M。

我们再用这个命令进入命令行详细看一下，这个命令进入后，输入top命令可以显示前10的内存分配，flat是堆栈中当前层的inuse内存值，cum是堆栈中本层级的累计inuse内存值。

可以看到，bytes.makeSlice这个内置方法竟然使用了24M内存，那么我们就考虑谁会用到makeslice呢，![heap_gdb](https://tva1.sinaimg.cn/large/006y8mN6gy1g98c5dze47j31fq0mu79j.jpg)

继续往下看，可以看到ReadFrom这个方法，看下源码，发现 ioutil.ReadAll() 里会调用 bytes.Buffer.ReadFrom, 

![image-20191123222106249](https://tva1.sinaimg.cn/large/006y8mN6gy1g98c67o6udj30hi0b40ur.jpg)

而 ReadFrom 会进行 makeSlice。让我们再回头看一下readAll的代码实现：

![image-20191123222129984](https://tva1.sinaimg.cn/large/006y8mN6gy1g98c6oud5sj30i709zq57.jpg)

这个函数主要作用就是从 io.Reader 里读取的数据放入 buffer 中，如果 buffer 空间不够，就按照每次 2x 所读取内容的大小+ MinRead 的算法递增，这里 MinRead 的大小是 512 个字节，也就是说如果我们一次性读取的内容过大，就会导致所使用的内存倍增，假设我们的所有任务文件总共有500M,那么所用的内存就有500M * 2 + 512B，所以我们分析很可能是这个原因导致的，那看看crawlab源码中是哪一段使用ReadAll读了文件，定位到了这里

![image-20191123222147655](https://tva1.sinaimg.cn/large/006y8mN6gy1g98c734tyjj30hq07dtae.jpg)     

这段代码直接将全部的文件内容，以二进制的形式读了进来，导致内存倍增，令人窒息的操作。至此，问题已经发现，那么如何去优化呢，其实在读大文件的时候，把文件内容全部读到内存，直接就翻车了，正确是处理方法有两种，一种是流式处理 ，如代码所示

![image-20191123222203534](https://tva1.sinaimg.cn/large/006y8mN6gy1g98c74vzzdj30dp09zwfp.jpg)

第二种方案就是分片处理，当读取的是二进制文件，没有换行符的时候，使用这种方案比较合适。

![image-20191123222215908](https://tva1.sinaimg.cn/large/006y8mN6gy1g98c7fsg21j30cx09z0u6.jpg)

我们这里采用的第二种方式来优化，优化后再来看下内存占用情况

我们采样程序运行30s的平均内存，并且在此期间访问上传文件的接口后，看到内存占用已经降到正常的水平了。

![image-20191123222231369](https://tva1.sinaimg.cn/large/006y8mN6gy1g98c7uw1iij30ou09f77c.jpg)v

刚刚我用一个实际案例介绍了如何去一步一步的分析及优化go程序的内存问题，大家可以在平时开发中多去探索，多使用。

接下来介绍一些go 开发中的小技巧，从细节上提高你的golang应用程序的性能。

众所周知Json 作为一种重要的数据格式，大家开发者经常用到。Go 语言里面原生支持了json的序列化以及反序列化，内部使用反射机制实现，我们都知道，反射的性能有点差，在高度依赖 json 解析的接口里，这往往会成为性能瓶颈，好在已有很多第三方库帮我们解决了这个问题，但是这么多库，到底要怎么选择呢，下面就给大家来一一分析一下，这里介绍4个json序列化反序列化的库,

![截屏2019-11-2322.23.03](https://tva1.sinaimg.cn/large/006y8mN6gy1g98c8lsud5j31uo0u0h6c.jpg)

看完这些你可能不知道到底该用哪一个，让我们来看下benchmark.

![image-20191123222330976](https://tva1.sinaimg.cn/large/006y8mN6gy1g98c8l87rlj30qo07m776.jpg)

- ​	•easyjson 无论是序列化还是反序列化都是最优的，序列化提升了1倍，反序列化提升了3倍

- ​	•jsoniter 性能也很好，接近于easyjson，关键是没有预编译过程，100%兼容原生库

- ​	•ffjson 的序列化提升并不明显，反序列化提升了1倍

- 所以综合考虑，建议大家使用 jsoniter，如果追求极致的性能，考虑 easyjson

  字符串拼接在开发中肯定是经常用到的，当你的应用中的某一部分大量用到字符串拼接，你最好留意一下下面的内容。我们列举4中字符串拼接的方式，看看你经常用的是哪个？

  ![截屏2019-11-2322.24.48](https://tva1.sinaimg.cn/large/006y8mN6gy1g98cab6m7nj31n30u0tmk.jpg)

第一种 + 号运算符，第二种是golang特有的一种方式，第三中是 strings.builder, 最后一种是strings.buffer。

看下benchmark你就会做出你最终的选择。性能最好的是strings.builder,而sprintf性能最差，+ 号运算符也没好到哪里去。

![image-20191123222511309](https://tva1.sinaimg.cn/large/006y8mN6gy1g98cabpls3j30qo07y764.jpg)

以上就是我今天分享的全部内容，希望能帮助到大家，下面是我的联系方式，有问题的可以线下继续交流。谢谢！

<hr />
