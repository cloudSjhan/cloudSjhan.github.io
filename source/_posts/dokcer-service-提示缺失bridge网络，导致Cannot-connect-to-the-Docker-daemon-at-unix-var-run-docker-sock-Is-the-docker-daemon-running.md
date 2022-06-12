---
title: >-
  dokcer.service 提示缺失bridge网络，导致Cannot connect to the Docker daemon at
  unix:///var/run/docker.sock. Is the docker daemon running?
tags: [docker]
copyright: true
date: 2019-06-04 15:25:01
permalink:
categories: docker
description:  dokcer.service 提示缺失bridge网络，导致Cannot connect to the Docker daemon at
  unix:///var/run/docker.sock. Is the docker daemon running?
image: https://static001.geekbang.org/resource/image/f6/bd/f6d81503fa575e72e5718649ca6289bd.jpg
---
<p class="description"></p>

<img src="https://" alt="" style="width:100%" />

<!-- more -->

操作过程：

1. 为CentOS7安装Docker，安装成功后，可以执行`docker`，但是`docker ps`等命令会报错：

2. ```shell
   Cannot connect to the Docker daemon at unix:///var/run/docker.sock. Is the docker daemon running?
   ```

   

分析：

一般这种错误都是由于操作者没有root权限，但是使用`sudo`执行也是同样的问题，这就纳闷了，没关系，看一下`docker.service`的执行日志：

```shell
systemctl status docker.service
```

发现有一句很重要的话：

```shell
Error starting daemon: Error initializing network controller: list bridge addresses failed: no available network

```

这是由于启动Docker的时候，默认的网络模式是桥接模式，这就需要向操作系统发送信号，让它帮我们建立一个`bridge`网络命名为`docker0`, 并且分配`172.17.0.1/16`。但是出于某种原因，该网络没有建立起来，我们只要手动执行这一系列操作就可以：

```shell
ip link add name docker0 type bridge

ip addr add dev docker0 172.17.0.1/16
```



最后重启`docker`:

```
systemclt restart docker
```



<hr />
