---
title: 自动化脚本实现go安装与升级
tags: [go]
copyright: true
date: 2019-01-12 23:55:47
permalink:
categories: go
description: 自动化脚本实现go安装与升级
image: https://static001.geekbang.org/resource/image/f7/00/f7343473fc0c358ded5b037189aa8d00.jpg
---
<p class="description"></p>

<img src="https://" alt="" style="width:100%" />

<!-- more -->

### 源码安装：

- 下载对应的操作系统的最新的 Golang 版本：https://golang.org/dl/

在 home 目录下建立  installGo目录，然后在该目录下新建升级与部署文件以及下载最新的 golang 源码包： 

以下是 installOrUpdate.sh 具体内容：

```shell
# !/bin/bash

if [ -z "$1" ]; then
​        echo "usage: ./install.sh go-package.tar.gz"
​        exit
fiif [ -d "/usr/local/go" ]; then
​        echo "Uninstalling old go version..."
​        sudo rm -rf /usr/local/go
fi
echo "Installing..."
sudo tar -C /usr/local -xzf $1
echo export GOPATH=/go" >> /etc/profile
echo export GOROOT=/usr/local/go >> /etc/profile
echo export PATH=$PATH:$GOROOT/bin:$GOPATH/bin1 >> /etc/profile
source /etc/profile
rm -rf $1
echo "Done"
```

然后运行：

sudo sh install.sh go1.10.linux-amd64.tar.gz

- 或者手动设置好GOPATH，GOROOT 

  编辑 /etc/profile 在文件尾部加入：

```shell
export GOPATH=/go
export GOROOT=/usr/local/go
export PATH=$PATH:$GOROOT/bin:$GOPATH/bin1

运行 source /etc/profile 让环境变量生效
```

## 至此，Go 已安装成功

- 升级

  如果需要升级的话只需要将最新的源码包下载到第一步的 installGo 文件夹下，然后运行sudo sh install.sh go1.xx.linux-amd64.tar.gz 即可。
- shell脚本更新地址[github](https://github.com/hantmac/Install_or_Update_Go_Automatically)

<hr />
