---
title: python 的logging模块实现json格式的日志输出
tags: [python]
copyright: true
date: 2018-09-27 16:28:21
permalink:
categories: python
description: 使用Python的内置logging实现json格式的日志输出
image: 
---
<p class="description"></p>

<img src="https://" alt="" style="width:100%" />

<!-- more -->

### 前言：

* 想要让开发过程或者是上线后的bug无处可藏，最好的方式便是在程序运行过程中，不断收集重要的日志，以供分析使用。Python中内置的log收集模块是logging，该模块使用起来比较方便，但是美中不足的地方就是日志的格式转成json比较麻烦。于是我结合logging和另一个模块[python-json-logger](https://github.com/madzak/python-json-logger)(pip install python-json-logger) ，实现json格式的日志输出。

### 源码：以下代码可以做成模块，直接导入使用

```Python
import logging, logging.config, os
import structlog
from structlog import configure, processors, stdlib, threadlocal
from pythonjsonlogger import jsonlogger
BASE_DIR = BASE_DIR = os.path.dirname(os.path.abspath(__file__))
DEBUG = True  # 标记是否在开发环境


# 给过滤器使用的判断
class RequireDebugTrue(logging.Filter):
    # 实现filter方法
    def filter(self, record):
        return DEBUG

def get_logger():
    LOGGING = {
    # 基本设置
        'version': 1,  # 日志级别
        'disable_existing_loggers': False,  # 是否禁用现有的记录器

    # 日志格式集合
        'formatters': {
        # 标准输出格式
            'json': {
            # [具体时间][线程名:线程ID][日志名字:日志级别名称(日志级别ID)] [输出的模块:输出的函数]:日志内容
                'format': '[%(asctime)s][%(threadName)s:%(thread)d][%(name)s:%(levelname)s(%(lineno)d)]\n[%(module)s:%(funcName)s]:%(message)s',
                'class': 'pythonjsonlogger.jsonlogger.JsonFormatter',
            }
        },
    # 过滤器
        'filters': {
            'require_debug_true': {
                '()': RequireDebugTrue,
            }
        },
    # 处理器集合
        'handlers': {
        # 输出到控制台
        # 输出到文件
            'TimeChecklog': {
                'level': 'DEBUG',
                'class': 'logging.handlers.RotatingFileHandler',
                'formatter': 'json',
                'filename': os.path.join("./log/", 'TimeoutCheck.log'),  # 输出位置
                'maxBytes': 1024 * 1024 * 5,  # 文件大小 5M
                'backupCount': 5,  # 备份份数
                'encoding': 'utf8',  # 文件编码
            },
        },
    # 日志管理器集合
        'loggers': {
        # 管理器
            'proxyCheck': {
                'handlers': ['TimeChecklog'],
                'level': 'DEBUG',
                'propagate': True,  # 是否传递给父记录器
            },
        }
    }

    configure(
        logger_factory=stdlib.LoggerFactory(),
        processors=[
            stdlib.render_to_log_kwargs]
    )


    logging.config.dictConfig(LOGGING)
    logger = logging.getLogger("proxyCheck")
    return logger


# 测试用例，你可以把get_logger()封装成一个模块，from xxx import get_logger()
logger1 = get_logger()
def test():
    try:
        a = 1 / 0
    except Exception as e:
        logger1.error(e)  # 写入错误日志
        #如果需要添加额外的信息，使用extra关键字即可
        logger1.error(e, extra={key1: value1, key2:value2})
        # 其他错误处理代码
        pass
test()
```



### ### 测试结果

* 测试的结果，可以在./log/xxx.log文件中看到输出的日志

```
{"asctime": "2018-09-28 09:52:12,622", "threadName": "MainThread", "thread": 4338656704, "name": "proxyCheck", "levelname": "ERROR", "%(lineno": null, "module": "mylog", "funcName": "test", "message": "division by zero"}

```

* 可以看到日志是json格式，这样你就可以很方便的使用grafna和ES将日志做成看板来展示了。



<hr />
