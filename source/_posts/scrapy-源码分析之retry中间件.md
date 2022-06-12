---
title: scrapy 源码分析之retry中间件与应用
tags: [scrapy,python]
copyright: true
date: 2019-03-29 19:33:40
permalink:
categories: 技术周刊
description: 分析scrapy的retry源码，定制化重试机制
image: https://ws1.sinaimg.cn/large/006tKfTcly1g1jwbqnvibj30go0dcgmw.jpg
---
<p class="description"></p>

<img src="https://" alt="" style="width:100%" />

<!-- more -->

`这次让我们分析scrapy重试机制的源码，学习其中的思想，编写定制化middleware,捕捉爬取失败的URL等信息。`

## scrapy简介

Scrapy是一个为了爬取网站数据，提取结构性数据而编写的应用框架。 可以应用在包括数据挖掘，信息处理或存储历史数据等一系列的程序中。

其最初是为了 [页面抓取](http://en.wikipedia.org/wiki/Screen_scraping) (更确切来说, [网络抓取](http://en.wikipedia.org/wiki/Web_scraping) )所设计的， 也可以应用在获取API所返回的数据(例如 [Amazon Associates Web Services](http://aws.amazon.com/associates/) ) 或者通用的网络爬虫。

一张图可看清楚scrapy中数据的流向：

![](https://scrapy-chs.readthedocs.io/zh_CN/0.24/_images/scrapy_architecture.png)

简单了解一下各个部分的功能，可以看下面简化版数据流：

![](https://ws2.sinaimg.cn/large/006tKfTcly1g1jwn7d37aj30tg0iy0v1.jpg)

##  总有漏网之鱼

不管你的主机配置多么吊炸天，还是网速多么给力，在scrapy的大规模任务中，最终爬取的item数量都不会等于期望爬取的数量，也就是说总有那么一些爬取失败的漏网之鱼，通过分析scrapy的日志，可以知道造成失败的原因有以下两种情况：

1. exception_count
2. httperror

![](https://ws1.sinaimg.cn/large/006tKfTcgy1g1jyf6bmcfj314w0kqq97.jpg)

以上的不管是exception还是httperror, scrapy中都有对应的retry机制，在`settings.py`文件中我们可以设置有关重试的参数，等运行遇到异常和错误时候，scrapy就会自动处理这些问题，其中最关键的部分就是重试中间件，下面让我们看一下scrapy的retry middleware。

##  RetryMiddle源码分析

在scrapy项目的`middlewares.py`文件中 敲如下代码：

```python
from scrapy.downloadermiddlewares.retry import RetryMiddleware
```

按住ctrl键（Mac是command键），鼠标左键点击RetryMiddleware进入该中间件所在的项目文件的位置，也可以通过查看文件的形式找到该该中间件的位置，路径是：

```shell
site-packages/scrapy/downloadermiddlewares/retry.RetryMiddleware
```

源码如下：

```Python
class RetryMiddleware(object):

    # IOError is raised by the HttpCompression middleware when trying to
    # decompress an empty response
    # 需要重试的异常状态，可以看出，其中有些是上面log中的异常
    EXCEPTIONS_TO_RETRY = (defer.TimeoutError, TimeoutError, DNSLookupError,
                           ConnectionRefusedError, ConnectionDone, ConnectError,
                           ConnectionLost, TCPTimedOutError, ResponseFailed,
                           IOError, TunnelError)

    def __init__(self, settings):
      # 读取 settings.py 中关于重试的配置信息，如果没有配置重试的话，直接跳过
        if not settings.getbool('RETRY_ENABLED'):
            raise NotConfigured
        self.max_retry_times = settings.getint('RETRY_TIMES')
        self.retry_http_codes = set(int(x) for x in settings.getlist('RETRY_HTTP_CODES'))
        self.priority_adjust = settings.getint('RETRY_PRIORITY_ADJUST')

    @classmethod
    def from_crawler(cls, crawler):
        return cls(crawler.settings)
# 如果response的状态码，是我们要重试的
    def process_response(self, request, response, spider):
        if request.meta.get('dont_retry', False):
            return response
        if response.status in self.retry_http_codes:
            reason = response_status_message(response.status)
            return self._retry(request, reason, spider) or response
        return response
# 出现了需要重试的异常状态，
    def process_exception(self, request, exception, spider):
        if isinstance(exception, self.EXCEPTIONS_TO_RETRY) \
                and not request.meta.get('dont_retry', False):
            return self._retry(request, exception, spider)
# 重试操作
    def _retry(self, request, reason, spider):
        retries = request.meta.get('retry_times', 0) + 1

        retry_times = self.max_retry_times

        if 'max_retry_times' in request.meta:
            retry_times = request.meta['max_retry_times']

        stats = spider.crawler.stats
        if retries <= retry_times:
            logger.debug("Retrying %(request)s (failed %(retries)d times): %(reason)s",
                         {'request': request, 'retries': retries, 'reason': reason},
                         extra={'spider': spider})
            retryreq = request.copy()
            retryreq.meta['retry_times'] = retries
            retryreq.dont_filter = True
            retryreq.priority = request.priority + self.priority_adjust

            if isinstance(reason, Exception):
                reason = global_object_name(reason.__class__)

            stats.inc_value('retry/count')
            stats.inc_value('retry/reason_count/%s' % reason)
            return retryreq
        else:
            stats.inc_value('retry/max_reached')
            logger.debug("Gave up retrying %(request)s (failed %(retries)d times): %(reason)s",
                         {'request': request, 'retries': retries, 'reason': reason},
                         extra={'spider': spider})
```

查看源码我们可以发现，对于返回http code的response，该中间件会通过process_response方法来处理，处理办法比较简单，判断response.status是否在retry_http_codes集合中，这个集合是读取的配置文件：

```python
RETRY_ENABLED = True                  # 默认开启失败重试，一般关闭
RETRY_TIMES = 3                         # 失败后重试次数，默认两次
RETRY_HTTP_CODES = [500, 502, 503, 504, 522, 524, 408]    # 碰到这些验证码，才开启重试
```

对于httperror的处理也是同样的道理，定义了一个 EXCEPTIONS_TO_RETRY的列表，里面存放所有的异常类型，然后判断传入的异常是否存在于该集合中，如果在就进入retry逻辑，不在就忽略。

## 源码思想的应用

了解scrapy如何处理异常后，就可以利用这种思想，写一个middleware，对爬取失败的漏网之鱼进行捕获，方便以后做补爬。

1. 在middlewares.py中 from scrapy.downloadermiddlewares.retry import RetryMiddleware, 写一个class，继承自RetryMiddleware；
2. 对父类的`process_response()`和`process_exception()`方法进行重写；
3. 将该middleware加入setting.py;
4. 注意事项：该中间件的Order_code不能过大，如果过大就会越接近下载器，就会优先于RetryMiddleware处理response，但这个中间件是用来处理最终的错误的，即当一个response 500进入中间件链路时，需要先经过retry中间件处理，不能先由我们写的中间件来处理，它不具有retry的功能，接收到500的response就直接放弃掉该request直接return了，这是不合理的。只有经过retry后仍然有异常的request才应当由我们写的中间件来处理，这时候你想怎么处理都可以，比如再次retry、return一个重新构造的response，但是如果你为了加快爬虫速度，不设置retry也是可以的。

Talk is cheap, show the code:

```Python
class GetFailedUrl(RetryMiddleware):
    def __init__(self, settings):
        self.max_retry_times = settings.getint('RETRY_TIMES')
        self.retry_http_codes = set(int(x) for x in settings.getlist('RETRY_HTTP_CODES'))
        self.priority_adjust = settings.getint('RETRY_PRIORITY_ADJUST')

    def process_response(self, request, response, spider):
        if response.status in self.retry_http_codes:
        # 将爬取失败的URL存下来，你也可以存到别的存储
            with open(str(spider.name) + ".txt", "a") as f:
                f.write(response.url + "\n")
            return response
        return response

    def process_exception(self, request, exception, spider):
    # 出现异常的处理
        if isinstance(exception, self.EXCEPTIONS_TO_RETRY):
            with open(str(spider.name) + ".txt", "a") as f:
                f.write(str(request) + "\n")
            return None
```

setting.py中添加该中间件：

```python
DOWNLOADER_MIDDLEWARES = {
   'myspider.middlewares.TabelogDownloaderMiddleware': 543,
    'myspider.middlewares.RandomProxy': 200,
    'myspider.middlewares.GetFailedUrl': 220,
}
```

为了测试，我们故意写错URL，或者将download_delay缩短，就会出现各种异常，但是我们现在能够捕获它们了：

![](https://ws2.sinaimg.cn/large/006tKfTcly1g1k09zlxtzj30ua076128.jpg)

   



<hr />