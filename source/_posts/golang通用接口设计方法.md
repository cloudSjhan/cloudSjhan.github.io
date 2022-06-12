---
title: golang中interface的通用设计方法
tags: [golang]
copyright: true
date: 2018-08-21 22:18:58
permalink:
categories: golang
description: golang中interface的通用设计方法
image:
---
<p class="description">golang中接口设计的通用方法</p>

<!-- more -->


```go
1. 接口定义
type XxxManager interface {
    Create(args argsType) (*XxxStruct, error)
    Get(args argsType) (**XxxStruct, error)
    Update(args argsType) (*XxxStruct, error)
    Delete(name string, options *DeleleOptions) error
}
2. 结构体定义 
type XxxManagerImpl struct {
    Name string
    Namespace string
    kubeCli *kubernetes.Clientset
}
3，构造函数
func NewXxxManagerImpl (namespace, name string, kubeCli *kubernetes.Clientset) XxxManager {
    return &XxxManagerImpl{
        Name name,
        Namespace namespace,
        kubeCli: kubeCli,
    }
}
4. 方法具体实现
func (xm *XxxManagerImpl) Create(args argsType) (*XxxStruct, error) {
    //具体的方法实现
}
```

**golang通用接口设计**

根据以上设计cdosapi封装接口：
