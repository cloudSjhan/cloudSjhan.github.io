---
title: Python中使用正则表达式的环视功能
tags: [正则表达式]
copyright: true
date: 2019-08-16 13:28:30
permalink:
categories: 正则表达式
description: 正则表达式中的环视功能解析以及在Python中的使用
image: https://static001.geekbang.org/resource/image/ba/83/ba97644fa167ef460bf349c092596a83.jpg
---
<p class="description"></p>
<img src="https://" alt="" style="width:100%" />

<!-- more -->

## 什么是环视

环视只是进行子表达式的匹配，并不占字符，匹配到的内容不保存，因此也叫做零宽断言，环视最终的匹配结果就是一个位置。

环视按照方向可以分为顺序环视和逆序环视两种，按是否进行匹配分为肯定和否定两种，组合起来就是四种模式。

| 环视表达式      | 解释                                           |
| --------------- | ---------------------------------------------- |
| (?=expression)  | 顺序肯定环视，表示所在位置右侧能匹配expression |
| (?!rexpression) | 顺序否定环视，表示所在位置右侧不匹配expression |
| (?<=expression) | 逆序肯定环视，表示所在位置左侧能匹配expression |
| (?<!expression) | 逆序否定环视，表示所在位置左侧不匹配expression |

只说概念可能有些抽象，分别举例子来演示一下具体的使用场景。

### 一些示例

1. 顺序肯定环视

```python
# s = 'xiaomi9iphone8iphone7',需要在每个手机型号后面加上逗号，变成 s= 'xiaomi9,iphone8,iphone7'
import re
print(re.sub(r'(?=iphone)',',',s))
# 顺序肯定环视，所确定的位置右边是字符串iPhone，在此位置即可添加逗号
```

2. 逆序肯定环视

```python
# s = 'Takes Reservations:No Delivery:No Take-out:Yes Accepts Credit Cards:Yes Good for Groups:No'
# 需求是要在Yes，和No的后面加上逗号，使之变成
# s = 'Takes Reservations:No, Delivery:No, Take-out:Yes, Accepts Credit Cards:Yes, Good for Groups:No'
import re
re.sub(r"(?<=(No))(?=(\s+))|(?<=(Yes))(?=(\s+))",',',s)
# 逆序肯定环视，所要确定的位左边必须能匹配上No,或者Yes
```

3. 顺序否定环视

```Python
# s = '123aaa',将s字符串变成 s='123,a,a,a,'
# 分析一下，就是在字符串右侧非数字的位置，添加逗号，即使用顺序否定环视，匹配右侧非数字位置
s = '123aaa'
re.sub(r'(?!\d+)',',',s)


```

4. 逆序否定环视

```python
# 将 s= 'aaa123'变成 s=  ',a,a,a,123'
# 分析一下，就是在非数字左侧的位置加逗号，使用逆序否定环视，匹配左侧非数字的位置
s= 'aaa123'
re.sub(r'(?<!\d)',',',b)
```

有些例子不是很合理，尽量能表达清楚环视的含义即可。



总结：

```markdown
环视的功能非常强大，也是正则中的一个难点，对于环视的理解，可以从应用和原理两个角度理解，如果想理解得更清晰、深入一些，还是从原理的角度理解好一些，正则匹配基本原理参考 NFA引擎匹配原理
```

<hr />
