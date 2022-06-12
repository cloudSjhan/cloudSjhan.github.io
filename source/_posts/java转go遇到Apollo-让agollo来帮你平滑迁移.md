---
title: java转go遇到Apollo?让agollo来帮你平滑迁移
tags: [golang]
copyright: true
date: 2020-04-17 15:50:33
permalink:
categories: golang
description: java 转 go 遇到 Apollo？ 让 agollo 来帮你平滑迁移
image:
---
<p class="description"></p>

<img src="https://" alt="" style="width:100%" />

<!-- more -->
![](https://tva1.sinaimg.cn/large/007S8ZIlgy1gdueckqd79j30u00u043g.jpg)

## Introduction

agollo 是[Apollo](https://github.com/ctripcorp/apollo)的 Golang 客户端

> Apollo（阿波罗）是携程框架部门研发的分布式配置中心，能够集中化管理应用不同环境、不同集群的配置，配置修改后能够实时推送到应用端，并且具备规范的权限、流程治理等特性，适用于微服务配置管理场景。

如果在使用 golang 重构 java 的过程中，使用到了分布式配置中心 Apollo，那么最快的方式就是使用原来的配置，保持最平滑的迁移，这个时候你就需要一个 Apollo 的 golang 客户端，agollo 可以是你的一个选择。

## 使用指南

### 1.1.环境要求

* Go 1.11+ (最好使用Go 1.12)

### 1.2.依赖

#### 1.2.1.使用 go get 方式

```
go get -u github.com/zouyx/agollo/v3@latest
```

#### 1.2.2.使用 go mod 方式

go.mod
```golang
require github.com/zouyx/agollo/v3 latest
```

执行
```
go mod tidy
```

import
```golang
import (
	"github.com/zouyx/agollo/v3"
)
```

FAQ
* [为什么要加+incompatible](https://github.com/golang/go/wiki/Modules#semantic-import-versioning)
* [go.mod不能下载agollo的解决方法](https://github.com/zouyx/agollo/issues/85)

### 1.3.必要设置

Apollo 客户端依赖于 AppId，Environment 等环境信息来工作，所以请确保阅读下面的说明并且做正确的配置：

* main ： your application
* app.properties (必要) ： 连接服务端必要配置
* seelog.xml（非必要）

#### 1.3.1.配置

加载优先级：

1. 类配置
2. 环境变量指定配置文件
3. 默认（app.properties）配置文件

##### 1.3.1.1.类配置

会覆盖`app.properties`中配置，在调用Start方法之前调用

``` golang
readyConfig:=&AppConfig{
		AppId:"test1",
		Cluster:"dev1",
		NamespaceName:"application1",
		Ip:"localhost:8889",
	}

	InitCustomConfig(func() (*AppConfig, error) {
		return readyConfig,nil
	})
```

类配置 agollo 的 demo:

```golang
package main

import (
	"fmt"
	"github.com/zouyx/agollo/v3"
	"github.com/zouyx/agollo/v3/env/config"
)

func InitAgolloConfig() error {
	readyConfig := &config.AppConfig{
		AppID:         "test",
		Cluster:       "default",
		NamespaceName: "application",
		IP:            "localhost:8080",
	}
	agollo.InitCustomConfig(func() (*config.AppConfig, error) {
		return readyConfig, nil
	})
	err := agollo.Start()
	if err != nil {
		fmt.Errorf("%v", err)
		return err
	}

	return nil
}

func main(){
    //Init Apollo
	err := config.InitAgolloConfig()
	if err != nil {
		fmt.Errorf("初始化Apollo失败", err)
		panic(err)
	}
    fmt.Println("初始化Apollo配置成功")

    //Use your apollo key to test
    value := agollo.GetStringValue("key", "")
    fmt.Println(value)
}

```

##### 1.3.1.2.环境变量指定配置文件

Linux/Mac
```bash
export AGOLLO_CONF=/a/conf.properties
```

Windows
```bash
set AGOLLO_CONF=c:/a/conf.properties
```

配置文件内容与app.properties内容一样

##### 1.3.1.3.文件配置 - app.properties

1.开发:请确保 **app.properties** 文件存在于workingdir目录下

2.打包后:请确保 **app.properties** 文件存在于与打包程序同级目录下，参考1.3.必要配置。

目前只支持json形式，其中字段包括：

* *appId* ：应用的身份信息，是从服务端获取配置的一个重要信息。
* *cluster* ：需要连接的集群，默认default
* *namespaceName* ：命名空间，默认：application（具体定义参考：[namespace](https://github.com/ctripcorp/apollo/wiki/Apollo核心概念之“Namespace”)）,**多namespace使用英文逗号分割**，
非key/value配置（json,properties,yml等），则配置为：namespace.文件类型。如：namespace.json

* *ip* ：Apollo的CONFIG_SERVICE的ip，非META_SERVICE地址

配置例子如下：

**一般配置**
```json
{
    "appId": "test",
    "cluster": "dev",
    "namespaceName": "application",
    "ip": "localhost:8888"
}
```

**多namespace配置**
```json
{
    "appId": "test",
    "cluster": "dev",
    "namespaceName": "application, applications1",
    "ip": "localhost:8888"
}
```

**非key/value namespace配置**
```json
{
    "appId": "test",
    "cluster": "dev",
    "namespaceName": "application.json,a.yml",
    "ip": "localhost:8888"
}
```

### 1.4日志组件

参考:

* [自定义日志组件](https://github.com/zouyx/agollo/wiki/%E8%87%AA%E5%AE%9A%E4%B9%89%E6%97%A5%E5%BF%97%E7%BB%84%E4%BB%B6)
* [使用seelog日志组件](https://github.com/zouyx/agollo/wiki/使用seelog日志组件)

## 启动方式

- 异步启动 agollo

场景：启动程序不依赖加载Apollo的配置。

``` go
func main() {
	 go agollo.Start()
}
```

- 同步启动 agollo（v1.2.0+）

场景：启动程序依赖加载 Apollo 的配置。例：初始化程序基础配置。

``` go
func main() {
	 agollo.Start()
}
```

- 启动 agollo - 自定义 logger 控件

``` go
func main() {
	 agollo.StartWithLogger(loggerInterface)
}
```

- 启动 agollo - 自定义 cache 控件 (v1.7.0+)

``` go
func main() {
	 agollo.StartWithCache(cacheInterface)
}
```

- 启动 agollo - 自定义各种控件 (v1.8.0+)

```go
func main() {
    agollo.SetLogger(loggerInterface)
	agollo.SetCache(cacheInterface)
	agollo.Start()
}
```

- 监听变更事件（阻塞）

``` go
func main() {
	event := agollo.ListenChangeEvent()
	changeEvent := <-event
	bytes, _ := json.Marshal(changeEvent)
	fmt.Println("event:", string(bytes))
}
```

## 基本方法

- String

```golang
agollo.GetStringValue(Key,DefaultValue)
```

- Int

```golang
agollo.GetIntValue(Key,DefaultValue)
```

- Float

```golang
agollo.GetFloatValue(Key,DefaultValue)
```

- Bool

```golang
agollo.GetBoolValue(Key,DefaultValue)
```



## 切换namespace获取配置

- 根据namespace获取配置

```golang
config := agollo.GetConfig(namespace)
```

- String

```golang
config.GetStringValue(Key,DefaultValue)
```

- Int

```golang
config.GetIntValue(Key,DefaultValue)
```

- Float

```golang
config.GetFloatValue(Key,DefaultValue)
```

- Bool

```golang
config.GetBoolValue(Key,DefaultValue)
```

## 自定义日志组件

复制以下代码至项目中，并在其中引用日志组件的方法进行打印 log

```go
type DefaultLogger struct {
}

func (this *DefaultLogger)Debugf(format string, params ...interface{})  {

}

func (this *DefaultLogger)Infof(format string, params ...interface{}) {

}


func (this *DefaultLogger)Warnf(format string, params ...interface{}) error {
	return nil
}

func (this *DefaultLogger)Errorf(format string, params ...interface{}) error {
	return nil
}


func (this *DefaultLogger)Debug(v ...interface{}) {

}
func (this *DefaultLogger)Info(v ...interface{}){

}

func (this *DefaultLogger)Warn(v ...interface{}) error{
	return nil
}

func (this *DefaultLogger)Error(v ...interface{}) error{
	return nil
}
```

启动

```go
func main() {
   agollo.SetLogger(&DefaultLogger{})
   agollo.Start()
}
```

## 自定义缓存组件

### 声明自定义缓存组件

```golang
//DefaultCache 默认缓存
type DefaultCache struct {
	defaultCache sync.Map
}

//Set 获取缓存
func (d *DefaultCache)Set(key string, value []byte, expireSeconds int) (err error)  {
	d.defaultCache.Store(key,value)
	return nil
}

//EntryCount 获取实体数量
func (d *DefaultCache)EntryCount() (entryCount int64){
	count:=int64(0)
	d.defaultCache.Range(func(key, value interface{}) bool {
		count++
		return true
	})
	return count
}

//Get 获取缓存
func (d *DefaultCache)Get(key string) (value []byte, err error){
	v, ok := d.defaultCache.Load(key)
	if !ok{
		return nil,errors.New("load default cache fail")
	}
	return v.([]byte),nil
}

//Range 遍历缓存
func (d *DefaultCache)Range(f func(key, value interface{}) bool){
	d.defaultCache.Range(f)
}

//Del 删除缓存
func (d *DefaultCache)Del(key string) (affected bool) {
	d.defaultCache.Delete(key)
	return true
}

//Clear 清除所有缓存
func (d *DefaultCache)Clear() {
	d.defaultCache=sync.Map{}
}

//DefaultCacheFactory 构造默认缓存组件工厂类
type DefaultCacheFactory struct {

}

//Create 创建默认缓存组件
func (d *DefaultCacheFactory) Create()CacheInterface {
	return &DefaultCache{}
}
```

### 使用自定义缓存

```golang
agollo.SetCache(&DefaultCacheFactory{})
agollo.Start()
```

----------------------

- 项目地址: https://github.com/zouyx/agollo



​                          *方资讯\*最新技术\*独家解读*

![](https://tva1.sinaimg.cn/large/00831rSTgy1gcmcur033tj306b06b74o.jpg)

<hr />
