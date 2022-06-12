---
title: Scrapy 解决URL被重定向无法抓取到数据问题301和302
tags: [Scrapy]
copyright: true
date: 2019-04-10 16:48:20
permalink:
categories: Scrapy
description: Scrapy 解决URL被重定向无法抓取到数据问题301和302
image: https://static001.geekbang.org/resource/image/56/39/56d49622a561b095ea59e77ba9168839.jpeg
---
<p class="description"></p>

<img src="https://" alt="" style="width:100%" />

<!-- more -->

1.什么是状态码301,302

301 Moved Permanently（永久重定向） 被请求的资源已永久移动到新位置，并且将来任何对此资源的引用都应该使用本响应返回的若干个URI之一。

解决（一）

1.在Request中将scrapy的dont_filter=True，因为scrapy是默认过滤掉重复的请求URL，添加上参数之后即使被重定向了也能请求到正常的数据了

## example

Request(url, callback=self.next_parse, dont_filter=True)
解决（二）

在scrapy框架中的 settings.py文件里添加

HTTPERROR_ALLOWED_CODES = [301]
解决（三）

使用requests模块遇到301和302问题时

def website():
    'url'
    headers = {'Accept':     'text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,*/*;q=0.8',
           'Accept-Encoding': 'gzip, deflate, sdch, br',
           'Accept-Language': 'zh-CN,zh;q=0.8',
           'Connection': 'keep-alive',
           'Host': 'pan.baidu.com',
           'Upgrade-Insecure-Requests': '1',
           'User-Agent': 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/55.0.2883.87 Safari/537.36'}

    url = 'https://www.baidu.com/'
    html = requests.get(url, headers=headers, allow_redirects=False)
    return html.headers['Location']
allow_redirects=False的意义为拒绝默认的301/302重定向从而可以通过html.headers[‘Location’]拿到重定向的URL。

<hr />
