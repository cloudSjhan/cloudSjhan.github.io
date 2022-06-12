---
title: CentOS7-安装docker-compose时由于pip10包管理导致的错误
tags: [Docker]
copyright: true
date: 2018-09-13 10:11:49
permalink:
categories: Docker
description: CentOS下安装Docker-compose时出现了 Cannot uninstall 'requests'. It is a distutils installed project and thus we cannot accurately determine which files belong to it which would lead to only a partial uninstall.
image:

---
<p class="description"></p>

<img src="https://" alt="" style="width:100%" />

<!-- more -->

* 今天在CentOS下安装docker-compose，遇到了Cannot uninstall 'requests'. It is a distutils installed project and thus we cannot accurately determine which files belong to it which would lead to only a partial uninstall.
  错误的原因是requests默认版本为2.6.0，但是docker-compose要2.9以上才支持，但是无法正常卸载2.9版本，是pip10对包的管理存在变化。
* 解决方案：
  * pip install -l requests==2.9

<hr />
