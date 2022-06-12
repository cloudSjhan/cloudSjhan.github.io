---
title: 技术周刊之解析Python中的赋值、浅拷贝、深拷贝
tags: [Python, 技术周刊]
copyright: true
date: 2018-09-09 14:39:05
permalink:
categories: 技术周刊
description: 这周让我们来看一下Python中关于赋值、浅拷贝、深拷贝的特性
image:
---
<p class="description"></p>

<img src="https://" alt="" style="width:100%" />

<!-- more -->

## 事情的起因

- 本周我们分享的主题是Python中关于浅拷贝和深拷贝的特性，想要深入研究Python中的浅拷贝和深拷贝的起因在于，我想生成一个json字符串，该字符串未dumps之前是一个Python的数据结构，里面包含字典，以及List，在遍历生成dictionary时候，出现一个bug，就是每次遍历生成的dictionary都是上一次的值，现象可以看以下代码。

  ```python
  # 这里我们定义一个函数get_data()
  def get_data():
     ...:     appid_dict = {}
     ...:     appid_all_dict = {}
     ...:     import pdb;pdb.set_trace()
     ...:     for i in range(10):
     ...:         appid_dict['a'] = i
     ...:         appid_all_dict[i] = appid_dict
  # 我们的初衷是想要得到
  # {0: {'a': 0}, 1: {'a': 1}, 2: {'a': 2}, 3: {'a': 3}}....这样的一个dict
  
  # 但是在调试过程中，发现得到的结果是这样的：
  # (Pdb) appid_all_dict
  # {0: {'a': 2}, 1: {'a': 2}, 2: {'a': 2}}
  # (Pdb) 
  # 即，后面的appid_dict都会把前面的覆盖掉，这是什么原因呢？
  # 我们这里先把原因说一下：因为Python中对dict的操作默认是浅拷贝，即同样的字典，使用多次的话，每次使用都是指向同一片内存地址(引用)，所以在上面的程序中后面对appid_dict的赋值，都将前面的给覆盖掉了，导致每一个appid_dict指向同一片内存，读取的当然就是最后一次的appid_dict的值，即上面程序的执行结果：
  {0: {'a': 9}, 1: {'a': 9}, 2: {'a': 9}, 3: {'a': 9}, 4: {'a': 9}, 5: {'a': 9}, 6: {'a': 9}, 7: {'a': 9}, 8: {'a': 9}, 9: {'a': 9}}
  
  
  
  ```

  - 那么如何修改这个bug，让程序输出我们想要得到的结果：

    ```json
    {0: {'a': 0}, 1: {'a': 1}, 2: {'a': 2}, 3: {'a': 3}, 4: {'a': 4}, 5: {'a': 5}, 6: {'a': 6}, 7: {'a': 7}, 8: {'a': 8}, 9: {'a': 9}}
    ```

  - 看完下面对于Python赋值、浅拷贝、深拷贝的解析，相信你就可以自己解决这个问题了

  #### Python中的赋值操作

  - 赋值：就是对象的引用
  - 举例： a = b: 赋值引用，a和b都指向同一个对象，如图所示![](https://ws4.sinaimg.cn/large/006tNbRwly1fv3bo527hfj30y80lwq7i.jpg)

  ## Python中浅拷贝

  - a = b.copy(): a 是b的浅拷贝，a和b是一个独立的对象，但是它们的子对象还是指向同一片引用。![](https://ws4.sinaimg.cn/large/006tNbRwly1fv3btp4y4ij30zs0neq9f.jpg)
  - Python中对字典的默认赋值操作就是浅拷贝，所以导致了文章开头所出现的情况。

  ## Python中的深拷贝

  - 首先import copy,导入copy模块（Python中自带），b = copy.deepcopy(a), 我们就说b是a的深拷贝，b拷贝了a所有的资源对象，并新开辟了一块地址空间，两者互不干涉。![](https://ws1.sinaimg.cn/large/006tNbRwly1fv3bymsju4j311w0oi100.jpg)

  ## 实际的例子来进一步说明

  ```python
  In [13]: import copy
  
  In [14]: def temp():
      ...:     a = [1, 2, 3, 4, ['a', 'b']]
      ...:     b = a # 赋值操作，直接传所有对象的引用
      ...:     c = copy.copy(a) # 浅拷贝，子对象指向同一引用
      ...:     d = copy.deepcopy(a) # 深拷贝，互不干涉
      ...:     a.append(5) # 修改对象a
      ...:     a[4].append('c') # 修改a中的数组
      ...:     print( 'a = ', a )
      ...:     print( 'b = ', b )
      ...:     print( 'c = ', c )
      ...:     print( 'd = ', d ) 
      ...:     
  
  In [15]: 
  
  In [15]: temp()
  a =  [1, 2, 3, 4, ['a', 'b', 'c'], 5]
  b =  [1, 2, 3, 4, ['a', 'b', 'c'], 5]
  c =  [1, 2, 3, 4, ['a', 'b', 'c']]
  d =  [1, 2, 3, 4, ['a', 'b']]
  
  ```

## 解决最初的问题

- 看到这里，我们再回头看文章最初的那个问题，就可以很easy地解决了。

  ```Python
  def get_data():
     ...:     appid_dict = {}
     ...:     appid_all_dict = {}
     ...:     import pdb;pdb.set_trace()
     ...:     for i in range(10):
          		appid_dict = copy.deepcopy(appid_dict)# 只需要加上这一行，使其成为深拷贝，问题解决！
     ...:         appid_dict['a'] = i
     ...:         appid_all_dict[i] = appid_dict
  ```

## 总结

要对Python的dictionary进行迭代分析，一定要注意其中的深拷贝问题，出现问题后，也要多往这方面考虑。

本期技术周刊到此结束。

![](https://timgsa.baidu.com/timg?image&quality=80&size=b9999_10000&sec=1536489838162&di=52a5d7c56631ad266740914505a80a32&imgtype=0&src=http%3A%2F%2Ffile.elecfans.com%2Fweb1%2FM00%2F57%2FB6%2Fo4YBAFtMadCAL43RAAHzi5GNn9o475.png)

<hr />
