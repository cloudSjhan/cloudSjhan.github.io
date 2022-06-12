---
title: centos7 编译、安装、配置、启动redis
tags: [redis]
copyright: true
date: 2019-08-02 17:00:22
permalink:
categories: redis
description: centos7 编译，安装，配置，启动Redis
image: https://user-gold-cdn.xitu.io/2018/7/20/164b5954ba398db7?imageView2/0/w/1280/h/960/format/webp/ignore-error/1
---
<p class="description"></p>
<img src="https://" alt="" style="width:100%" />

<!-- more -->

1. 下载Redis的安装包

   wget http://download.redis.io/releases/redis-4.0.6.tar.gz

2. 解压

   tar -zxvf redis-4.0.6.tar.gz

3. cd redis-4.0.6

4. 编译

   make MALLOC=libc

5. 环境配置，将配置文件以你想暴露的Redis为名，复制到/etc/redis

   mkdir /etc/redis(需要root权限)

   cp redis.conf /etc/redis/6379.conf

6. cd utils & vim redis_init_script

   修改 redis_init_script中如图所示的部分

   [](http://ww1.sinaimg.cn/large/006tNc79gy1g5lgb221p3j316w0j619h.jpg)

   配置chkconfig是为了开机配置开机自启Redis，下面的横线部分换成你自己的Redis安装路径，可选择自己想要暴露的端口

7. 将修改后的redis_init_script拷贝至开机启动目录，并修改文件名为redisd, 一般后台服务都已d结尾

   cp redis_init_script /etc/init.d/redisd

8. 设置开机启动

   chkconfig redisd on

   如果之前没有在redis_init_script中添加chkconfig,就会提示不支持该命令

9. service redisd start 启动Redis

10. 后台启动的话，service redis start &

    

<hr />
