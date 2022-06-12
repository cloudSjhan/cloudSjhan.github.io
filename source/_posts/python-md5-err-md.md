---
title: python3中遇到'TypeError Unicode-objects must be encoded before hashing'
tags: [python, md5编码]
copyright: true
date: 2018-08-20 22:18:58
permalink:
categories: python
description: Python中进行md5加密时遇到的编码问题
image:
---
<p class="description">Python3中进行MD5加密，遇到编码问题</p>

<!-- more -->

```python
import hashlib
from urllib.parse import urlencode, quote_plus
import urllib



def verfy_ac(private_key):

    item = {
     "Action"     :  "CreateUHostInstance",
     "CPU"        :  2,
     "ChargeType" :  "Month",
     "DiskSpace"  :  10,
     "ImageId"    :  "f43736e1-65a5-4bea-ad2e-8a46e18883c2",
     "LoginMode"  :  "Password",
     "Memory"     :  2048,
     "Name"       :  "Host01",
     "Password"   :  "VUNsb3VkLmNu",
     "PublicKey"  :  "ucloudsomeone%40example.com1296235120854146120",
     "Quantity"   :  1,
     "Region"     :  "cn-bj2",
     "Zone"       :  "cn-bj2-04"
 }
    # 将参数串排序

    params_data = ""
    import pdb;pdb.set_trace()
    for key, value in item.items():
        params_data = params_data + str(key) + str(value)
    params_data = params_data + private_key
    params_data_en = quote_plus(params_data)

    sign = hashlib.sha1()
    sign.update(params_data_en.encode('utf8'))
    signature = sign.hexdigest()

    return signature


print(verfy_ac("46f09bb9fab4f12dfc160dae12273d5332b5debe"))

```

这是[ucloud官方的API教程](https://docs.ucloud.cn/api/summary/signature)，想根据此教程生成签名，教程中的代码是基于Python2.7编写，我将其改成了Python3.但是在执行时报错：

```python
TypeError: Unicode-objects must be encoded before hashing

```

------

排错后发现python3中字符对象是unicode对象，不能直接加密，需要编码后才能进行update。

就是改成如下即可：

```python
sign.update(params_data_en.encode('utf8'))
```





<hr />
