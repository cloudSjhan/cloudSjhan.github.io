---
title: 常用的Python小模块
tags: [python]
copyright: true
date: 2018-09-06 16:24:41
permalink:
categories: python
description: 常用的Python模块，即查即用
image:
---
<p class="description"></p>

<img src="https://" alt="" style="width:100%" />

<!-- more -->

* 工作或者生活中总会遇到一些常用的Python模块，为了避免重复的工作，将这些自己写过的Python模块记录下来，方便使用的时候查找。

### Python写CSV文件，并防止中文乱码

```python
def write_csv(a_list,b_list):
    with open('vm_data.csv', 'w') as f:
        f.write(codecs.BOM_UTF8.decode())
        writer1 = csv.writer(f,  dialect='excel')
        #写CVS的标题
        writer1.writerow(['a', 'b'])
        #将数据写入CSV文件
        writer1.writerows(zip(a_list, b_list))

```



### Python将数据结构转为json,并优化json字符串的结构，处理中文乱码

```python
    with open("appid.json", 'w', encoding='utf8', ) as f:
        f.write(json.dumps(final, sort_keys=True, indent=2, ensure_ascii=False))
    # sort_keys = True: 将字典的key按照字母排序
    # ident = 2: 优化json字符串结构，看起来更美观
    # ensure_ascii=False: 防止json字符串中的中文乱码
```



### 使用requests包进行网络请求（以post为例）

```
def  get_data(url):
    final = {}
    url = "http://xxxx.com"
    request_body = {
        'access_token': access_token,
        'request_body': {"params1": param1, 'params2': param2}
    }
    headers = {
        'Content-type': 'application/json'
    }
    data = requests.post(url, headers=headers, data=json.dumps(request_body))
```



<hr />
