---
title: python 发送各种格式的邮件
tags: [python]
copyright: true
date: 2018-09-17 10:55:56
permalink:
categories: python
description: 使用Python发送各种格式的邮件
image:
---
<p class="description"></p>

<img src="https://" alt="" style="width:100%" />

<!-- more -->

```Python
import smtplib
from email.mime.multipart import MIMEMultipart
from email.mime.text import MIMEText
from email.mime.application import MIMEApplication
_user = "sigeken@qq.com"
_pwd  = "***"
_to   = "402363522@qq.com"
 
#如名字所示Multipart就是分多个部分
msg = MIMEMultipart()
msg["Subject"] = "don't panic"
msg["From"]    = _user
msg["To"]      = _to
 
#---这是文字部分---
part = MIMEText("乔装打扮，不择手段")
msg.attach(part)
 
#---这是附件部分---
#xlsx类型附件
part = MIMEApplication(open('foo.xlsx','rb').read())
part.add_header('Content-Disposition', 'attachment', filename="foo.xlsx")
msg.attach(part)
 
#jpg类型附件
part = MIMEApplication(open('foo.jpg','rb').read())
part.add_header('Content-Disposition', 'attachment', filename="foo.jpg")
msg.attach(part)
 
#pdf类型附件
part = MIMEApplication(open('foo.pdf','rb').read())
part.add_header('Content-Disposition', 'attachment', filename="foo.pdf")
msg.attach(part)
 
#mp3类型附件
part = MIMEApplication(open('foo.mp3','rb').read())
part.add_header('Content-Disposition', 'attachment', filename="foo.mp3")
msg.attach(part)
 
s = smtplib.SMTP("smtp.qq.com", timeout=30)#连接smtp邮件服务器,端口默认是25
s.login(_user, _pwd)#登陆服务器
s.sendmail(_user, _to, msg.as_string())#发送邮件
s.close()
```



<hr />
