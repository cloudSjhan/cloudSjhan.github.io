---
title: postman 接口测试神器
tags: [postman,工具]
copyright: true
date: 2018-12-08 23:41:16
permalink:
categories: 工具
description: postman接口测试神器使用技巧
image: https://ws3.sinaimg.cn/large/006tNbRwly1fxzrukfk4nj3074074748.jpg
---
<p class="description"></p>

<!-- more -->

# [Postman 接口测试神器](https://malun666.github.io/aicoder_vip_doc/#/pages/postman?id=postman-%e6%8e%a5%e5%8f%a3%e6%b5%8b%e8%af%95%e7%a5%9e%e5%99%a8)

Postman 是一个接口测试和 http 请求的神器，非常好用。

官方 github 地址: <https://github.com/postmanlabs>

Postman 的优点：

- 支持各种的请求类型: get、post、put、patch、delete 等
- 支持在线存储数据，通过账号就可以进行迁移数据
- 很方便的支持请求 header 和请求参数的设置
- 支持不同的认证机制，包括 Basic Auth，Digest Auth，OAuth 1.0，OAuth 2.0 等
- 响应数据是自动按照语法格式高亮的，包括 HTML，JSON 和 XML

>  以下内容主要参考： [Github: api_tool_postman](https://github.com/crifan/api_tool_postman)  

## [安装](https://malun666.github.io/aicoder_vip_doc/#/pages/postman?id=%e5%ae%89%e8%a3%85)

Postman 可以单独作为一个应用安装，也可以作为 chrome 的一个插件安装。

-  chrome 插件安装, [Postman 插件地址](https://chrome.google.com/webstore/detail/postman/fhbjgbiflinjbdggehcddcbncdddomop)  
-  [单独应用安装下载](http://files.cnblogs.com/files/mafly/postman-4.1.2.rar)  

下面主要介绍下载安装独立版本app 软件的 Postman 的过程：

去主页[Postman 官网](https://www.getpostman.com/)找到：[Postman | Apps](https://www.getpostman.com/apps)

![img](https://ask.qcloudimg.com/http-save/yehe-1171488/c7j24uyvhy.png?imageView2/2/w/1620)

去下载自己平台的版本：

- Mac
- Windows（x86/x64）
- Linux（x86/x64） 即可。

## [快速入门，总体使用方略](https://malun666.github.io/aicoder_vip_doc/#/pages/postman?id=%e5%bf%ab%e9%80%9f%e5%85%a5%e9%97%a8%ef%bc%8c%e6%80%bb%e4%bd%93%e4%bd%bf%e7%94%a8%e6%96%b9%e7%95%a5)

安装成功后，打开软件。

### [新建接口](https://malun666.github.io/aicoder_vip_doc/#/pages/postman?id=%e6%96%b0%e5%bb%ba%e6%8e%a5%e5%8f%a3)

对应的Request：`New -> Request`

![img](https://ask.qcloudimg.com/http-save/yehe-1171488/a43avzlvvt.png?imageView2/2/w/1620)

或，在右边的 Tab 页面中点击加号+：

![img](https://ask.qcloudimg.com/http-save/yehe-1171488/kr7zu3lg54.png?imageView2/2/w/1620)

即可看到新建的 Tab 页：

![img](https://ask.qcloudimg.com/http-save/yehe-1171488/7x0f5ohfxq.png?imageView2/2/w/1620)

### [设置 HTTP 请求的方法](https://malun666.github.io/aicoder_vip_doc/#/pages/postman?id=%e8%ae%be%e7%bd%ae-http-%e8%af%b7%e6%b1%82%e7%9a%84%e6%96%b9%e6%b3%95)

设置 HTTP 的 Method 方法和输入 api 的地址

![img](https://ask.qcloudimg.com/http-save/yehe-1171488/nvkjcxavj1.png?imageView2/2/w/1620)

### [设置相关请求头信息](https://malun666.github.io/aicoder_vip_doc/#/pages/postman?id=%e8%ae%be%e7%bd%ae%e7%9b%b8%e5%85%b3%e8%af%b7%e6%b1%82%e5%a4%b4%e4%bf%a1%e6%81%af)

![img](https://ask.qcloudimg.com/http-save/yehe-1171488/sde0dugknd.png?imageView2/2/w/1620)

![img](https://ask.qcloudimg.com/http-save/yehe-1171488/d4vmkb97c4.png?imageView2/2/w/1620)

### [设置相关 GET 或 POST 等的参数](https://malun666.github.io/aicoder_vip_doc/#/pages/postman?id=%e8%ae%be%e7%bd%ae%e7%9b%b8%e5%85%b3-get-%e6%88%96-post-%e7%ad%89%e7%9a%84%e5%8f%82%e6%95%b0)

![img](https://ask.qcloudimg.com/http-save/yehe-1171488/qgi9b1hmbs.png?imageView2/2/w/1620)

### [发送请求](https://malun666.github.io/aicoder_vip_doc/#/pages/postman?id=%e5%8f%91%e9%80%81%e8%af%b7%e6%b1%82)

都填写好之后，点击 Send 去发送请求 Request：

![img](https://ask.qcloudimg.com/http-save/yehe-1171488/u4cd93numl.png?imageView2/2/w/1620)

### [查看响应 Response的信息](https://malun666.github.io/aicoder_vip_doc/#/pages/postman?id=%e6%9f%a5%e7%9c%8b%e5%93%8d%e5%ba%94-response%e7%9a%84%e4%bf%a1%e6%81%af)

![img](https://ask.qcloudimg.com/http-save/yehe-1171488/oo22jbrg8v.png?imageView2/2/w/1620)

然后可以重复上述修改 Request 的参数，点击 Send 去发送请求的过程，以便调试到 API 接口正常工作为止。

### [保存接口配置](https://malun666.github.io/aicoder_vip_doc/#/pages/postman?id=%e4%bf%9d%e5%ad%98%e6%8e%a5%e5%8f%a3%e9%85%8d%e7%bd%ae)

待整个接口都调试完毕后，记得点击 Save 去保存接口信息：

![img](https://ask.qcloudimg.com/http-save/yehe-1171488/n2p3eq0ie6.png?imageView2/2/w/1620)

去保存当前 API 接口，然后需要填写相关的接口信息：

- Request Name: 请求的名字 
  - 我一般习惯用保存为 接口的最后的字段名，比如`http://{% raw %}{{% endraw %}{server_address}}/ucows/login/login`中的`/login/login`
- Request Description: 接口的描述 
  - `可选` 最好写上该接口的要实现的基本功能和相关注意事项
  - 支持 Markdown 语法
- Select a collection or folder to save: 选择要保存到哪个分组（或文件夹） 
  - 往往保存到某个 API 接口到所属的该项目名的分组

![img](https://ask.qcloudimg.com/http-save/yehe-1171488/zntknyfx6w.png?imageView2/2/w/1620)

填写好内容，选择好分组，再点击保存：

![img](https://ask.qcloudimg.com/http-save/yehe-1171488/sb5yb8cwpi.png?imageView2/2/w/1620)

此时，Tab 的右上角的黄色点（表示没有保存）消失了，表示已保存。

且对应的分组中可以看到对应的接口了：

![img](https://ask.qcloudimg.com/http-save/yehe-1171488/x3lmlaykbk.png?imageView2/2/w/1620)

>  [warning] 默认不保存返回的 Response 数据  

- 直接点击 Save 去保存，只能保存 API 本身（的 Request 请求），不会保存 Response 的数据
- 想要保存 Response 数据，需要用后面要介绍的 [多个 Example](http://book.crifan.com/books/api_tool_postman/website/postman_func_resp/save_multi_example.html)

## [Request 的多参数操作详解](https://malun666.github.io/aicoder_vip_doc/#/pages/postman?id=request-%e7%9a%84%e5%a4%9a%e5%8f%82%e6%95%b0%e6%93%8d%e4%bd%9c%e8%af%a6%e8%a7%a3)

### [自动解析多个参数 Params](https://malun666.github.io/aicoder_vip_doc/#/pages/postman?id=%e8%87%aa%e5%8a%a8%e8%a7%a3%e6%9e%90%e5%a4%9a%e4%b8%aa%e5%8f%82%e6%95%b0-params)

比如，对于一个 GET 的请求的 url 是： `http://openapi.youdao.com/api?q=纠删码(EC)的学习&from=zh_CHS&to=EN&appKey=152e0e77723a0026&salt=4&sign=6BE15F1868019AD71C442E6399DB1FE4`

对应着其实是`?key=value`形式中包含多个 Http 的 GET 的 query string=query parameters

Postman 可以自动帮我们解析出对应参数，可以点击 Params：

![img](https://ask.qcloudimg.com/http-save/yehe-1171488/9kga5uhyuq.png?imageView2/2/w/1620)

看到展开的多个参数：

![img](https://ask.qcloudimg.com/http-save/yehe-1171488/2agjjtnusz.png?imageView2/2/w/1620)

如此就可以很方便的修改，增删对应的参数了。

### [临时禁用参数](https://malun666.github.io/aicoder_vip_doc/#/pages/postman?id=%e4%b8%b4%e6%97%b6%e7%a6%81%e7%94%a8%e5%8f%82%e6%95%b0)

且还支持，在不删除某参数的情况下，如果想要暂时不传参数，可以方便的通过不勾选的方式去实现：

![img](https://ask.qcloudimg.com/http-save/yehe-1171488/atacmafooc.png?imageView2/2/w/1620)

### [批量编辑 GET 的多个参数](https://malun666.github.io/aicoder_vip_doc/#/pages/postman?id=%e6%89%b9%e9%87%8f%e7%bc%96%e8%be%91-get-%e7%9a%84%e5%a4%9a%e4%b8%aa%e5%8f%82%e6%95%b0)

当然，如果想要批量的编辑参数，可以点击右上角的Bulk Edit，去实现批量编辑。

![img](https://ask.qcloudimg.com/http-save/yehe-1171488/xn3piv6qv8.png?imageView2/2/w/1620)

## [接口描述与自动生成文档](https://malun666.github.io/aicoder_vip_doc/#/pages/postman?id=%e6%8e%a5%e5%8f%a3%e6%8f%8f%e8%bf%b0%e4%b8%8e%e8%87%aa%e5%8a%a8%e7%94%9f%e6%88%90%e6%96%87%e6%a1%a3)

API 的描述中，也支持 Markdown，官方的接口说明文档：[Intro to API documentation](https://www.getpostman.com/docs/postman/api_documentation/intro_to_api_documentation)。

所以，可以很方便的添加有条理的接口描述，尤其是参数解释了：

![img](https://ask.qcloudimg.com/http-save/yehe-1171488/428i64obvu.png?imageView2/2/w/1620)

### [描述支持 markdown 语法](https://malun666.github.io/aicoder_vip_doc/#/pages/postman?id=%e6%8f%8f%e8%bf%b0%e6%94%af%e6%8c%81-markdown-%e8%af%ad%e6%b3%95)

![img](https://ask.qcloudimg.com/http-save/yehe-1171488/gae7gd5ldb.png?imageView2/2/w/1620)

而对于要解释的参数，可以通过之前的`Param -> Bulk Edit`的内容：

![img](https://ask.qcloudimg.com/http-save/yehe-1171488/k8h8cccbmy.png?imageView2/2/w/1620)

拷贝过来，再继续去编辑：

![img](https://ask.qcloudimg.com/http-save/yehe-1171488/q3ehctbg4b.png?imageView2/2/w/1620)

以及添加更多解释信息：

![img](https://ask.qcloudimg.com/http-save/yehe-1171488/cvxks97jg5.png?imageView2/2/w/1620)

点击 Update 后，即可保存。

### [发布接口并生成 markdown 的描述文件](https://malun666.github.io/aicoder_vip_doc/#/pages/postman?id=%e5%8f%91%e5%b8%83%e6%8e%a5%e5%8f%a3%e5%b9%b6%e7%94%9f%e6%88%90-markdown-%e7%9a%84%e6%8f%8f%e8%bf%b0%e6%96%87%e4%bb%b6)

去发布后：

![img](https://ask.qcloudimg.com/http-save/yehe-1171488/6td22h0mcp.png?imageView2/2/w/1620)

对应的效果：[有道翻译](https://documenter.getpostman.com/view/669382/collection/77fd4ek)

![img](https://ask.qcloudimg.com/http-save/yehe-1171488/w9r9f2lhvj.png?imageView2/2/w/1620)

![img](https://ask.qcloudimg.com/http-save/yehe-1171488/6w93zr891f.png?imageView2/2/w/1620)

## [Response 深入](https://malun666.github.io/aicoder_vip_doc/#/pages/postman?id=response-%e6%b7%b1%e5%85%a5)

### [Response 数据显示模式](https://malun666.github.io/aicoder_vip_doc/#/pages/postman?id=response-%e6%95%b0%e6%8d%ae%e6%98%be%e7%a4%ba%e6%a8%a1%e5%bc%8f)

Postman 对于返回的 Response 数据，支持三种显示模式。

- `默认`格式化后的 Pretty 模式

![img](https://ask.qcloudimg.com/http-save/yehe-1171488/90i3wajcea.png?imageView2/2/w/1620)

- Raw 原始模式

点击Raw，可以查看到返回的没有格式化之前的原始数据：

![img](https://ask.qcloudimg.com/http-save/yehe-1171488/2o5e02t9ac.png?imageView2/2/w/1620)

- Preview 预览模式

以及 Preview，是对应 Raw 原始格式的预览模式：

![img](https://ask.qcloudimg.com/http-save/yehe-1171488/jbnnfhoye4.png?imageView2/2/w/1620)

Preview 这种模式的显示效果，好像是对于返回的是 html 页面这类，才比较有效果。

### [Response 的 Cookies](https://malun666.github.io/aicoder_vip_doc/#/pages/postman?id=response-%e7%9a%84-cookies)

很多时候普通的 API 调用，倒是没有 Cookie 的：

![img](https://ask.qcloudimg.com/http-save/yehe-1171488/1srkh454hi.png?imageView2/2/w/1620)

### [Response 的 Headers 头信息](https://malun666.github.io/aicoder_vip_doc/#/pages/postman?id=response-%e7%9a%84-headers-%e5%a4%b4%e4%bf%a1%e6%81%af)

举例，此处返回的是有 Headers 头信息的：

![img](https://ask.qcloudimg.com/http-save/yehe-1171488/4hjkacbfao.png?imageView2/2/w/1620)

可以从中看到服务器是 Nginx 的。

## [保存多个 Example](https://malun666.github.io/aicoder_vip_doc/#/pages/postman?id=%e4%bf%9d%e5%ad%98%e5%a4%9a%e4%b8%aa-example)

之前想要实现，让导出的 API 文档中能看到接口返回的 Response 数据。后来发现是Example这个功能去实现此效果的。

### [如何添加 Example](https://malun666.github.io/aicoder_vip_doc/#/pages/postman?id=%e5%a6%82%e4%bd%95%e6%b7%bb%e5%8a%a0-example)

![img](https://ask.qcloudimg.com/http-save/yehe-1171488/4wjzn3wjr5.png?imageView2/2/w/1620)

继续点击Save Example：

![img](https://ask.qcloudimg.com/http-save/yehe-1171488/pyzrhod8jg.png?imageView2/2/w/1620)

保存后，就能看到Example(1)了：

![img](https://ask.qcloudimg.com/http-save/yehe-1171488/dhmfi3mz78.png?imageView2/2/w/1620)

### [单个 Example 在导出的 API 文档中的效果](https://malun666.github.io/aicoder_vip_doc/#/pages/postman?id=%e5%8d%95%e4%b8%aa-example-%e5%9c%a8%e5%af%bc%e5%87%ba%e7%9a%84-api-%e6%96%87%e6%a1%a3%e4%b8%ad%e7%9a%84%e6%95%88%e6%9e%9c)

然后再去导出文档，导出文档中的确能看到返回数据的例子： 

![img](https://ask.qcloudimg.com/http-save/yehe-1171488/651td0ljtp.png?imageView2/2/w/1620)

### [多个 Example 在导出的 API 文档中的效果](https://malun666.github.io/aicoder_vip_doc/#/pages/postman?id=%e5%a4%9a%e4%b8%aa-example-%e5%9c%a8%e5%af%bc%e5%87%ba%e7%9a%84-api-%e6%96%87%e6%a1%a3%e4%b8%ad%e7%9a%84%e6%95%88%e6%9e%9c)

![img](https://ask.qcloudimg.com/http-save/yehe-1171488/6bte63z6hq.png?imageView2/2/w/1620)

![img](https://ask.qcloudimg.com/http-save/yehe-1171488/6ckvn8fm3x.png?imageView2/2/w/1620)

## [其他好用的功能及工具](https://malun666.github.io/aicoder_vip_doc/#/pages/postman?id=%e5%85%b6%e4%bb%96%e5%a5%bd%e7%94%a8%e7%9a%84%e5%8a%9f%e8%83%bd%e5%8f%8a%e5%b7%a5%e5%85%b7)

### [分组 Collection](https://malun666.github.io/aicoder_vip_doc/#/pages/postman?id=%e5%88%86%e7%bb%84-collection)

在刚开始一个项目时，为了后续便于组织和管理，把同属该项目的多个 API，放在一组里

所以要先去新建一个 Collection: `New -> Collection`

![img](https://ask.qcloudimg.com/http-save/yehe-1171488/lokimruefe.png?imageView2/2/w/1620)

使用了段时间后，建了多个分组的效果：

![img](https://ask.qcloudimg.com/http-save/yehe-1171488/ggk7qt7s4f.png?imageView2/2/w/1620)

单个分组展开后的效果：

![img](https://ask.qcloudimg.com/http-save/yehe-1171488/w64q4ziv08.png?imageView2/2/w/1620)

### [历史记录 History](https://malun666.github.io/aicoder_vip_doc/#/pages/postman?id=%e5%8e%86%e5%8f%b2%e8%ae%b0%e5%bd%95-history)

Postman 支持 history 历史记录，显示出最近使用过的 API： 

![img](https://ask.qcloudimg.com/http-save/yehe-1171488/ui0g94c42i.png?imageView2/2/w/1620)

### [用环境变量实现多服务器版本](https://malun666.github.io/aicoder_vip_doc/#/pages/postman?id=%e7%94%a8%e7%8e%af%e5%a2%83%e5%8f%98%e9%87%8f%e5%ae%9e%e7%8e%b0%e5%a4%9a%e6%9c%8d%e5%8a%a1%e5%99%a8%e7%89%88%e6%9c%ac)

#### [现存问题](https://malun666.github.io/aicoder_vip_doc/#/pages/postman?id=%e7%8e%b0%e5%ad%98%e9%97%ae%e9%a2%98)

在测试 API 期间，往往存在多种环境，对应 IP 地址（或域名也不同）

比如：

- Prod: 

  ```
  http://116.62.25.57/ucows
  ```

  - 用于开发完成发布到生产环境

- Dev: 

  ```
  http://123.206.191.125/ucows
  ```

  - 用于开发期间的线上的 Development 的测试环境

- LocalTest: 

  ```
  http://192.168.0.140:80/ucows
  ```

  - 用于开发期间配合后台开发人员的本地局域网内的本地环境，用于联合调试 API 接口

而在测试 API 期间，往往需要手动去修改 API 的地址：

![img](https://ask.qcloudimg.com/http-save/yehe-1171488/l7grallxhf.png?imageView2/2/w/1620)

效率比较低，且地址更换后之前地址就没法保留了。

另外，且根据不同 IP 地址（或者域名）也不容易识别是哪套环境。

### [Postman 支持用 Environment 环境变量去实现多服务器版本](https://malun666.github.io/aicoder_vip_doc/#/pages/postman?id=postman-%e6%94%af%e6%8c%81%e7%94%a8-environment-%e7%8e%af%e5%a2%83%e5%8f%98%e9%87%8f%e5%8e%bb%e5%ae%9e%e7%8e%b0%e5%a4%9a%e6%9c%8d%e5%8a%a1%e5%99%a8%e7%89%88%e6%9c%ac)

后来发现 Postman 中，有 Environment 和 Global Variable，用于解决这个问题，实现不同环境的管理：

![img](https://ask.qcloudimg.com/http-save/yehe-1171488/rzkb6ahdi2.png?imageView2/2/w/1620)

>  很明显，就可以用来实现不用手动修改 url 中的服务器地址，从而动态的实现，支持不同服务器环境:  

- Production 生产环境
- Development 开发环境
- Local 本地局域网环境

#### [如何使用 Enviroment 实现多服务器版本](https://malun666.github.io/aicoder_vip_doc/#/pages/postman?id=%e5%a6%82%e4%bd%95%e4%bd%bf%e7%94%a8-enviroment-%e5%ae%9e%e7%8e%b0%e5%a4%9a%e6%9c%8d%e5%8a%a1%e5%99%a8%e7%89%88%e6%9c%ac)

![img](https://ask.qcloudimg.com/http-save/yehe-1171488/pvj87t9bl9.png?imageView2/2/w/1620)

或者：

![img](https://ask.qcloudimg.com/http-save/yehe-1171488/4cqehxbdq1.png?imageView2/2/w/1620)

![img](https://ask.qcloudimg.com/http-save/yehe-1171488/t6gzc4ktvw.png?imageView2/2/w/1620)

>  Environments are a group of variables & values, that allow you to quickly switch the context for your requests and collections.  Learn more about environments  You can declare a variable in an environment and give it a starting value, then use it in a request by putting the variable name within curly-braces. Create an environment to get started.  

输入 Key 和 value：

![img](https://ask.qcloudimg.com/http-save/yehe-1171488/tdph8jo5u3.png?imageView2/2/w/1620)

点击 Add 后：

![img](https://ask.qcloudimg.com/http-save/yehe-1171488/nvb1s1ht5r.png?imageView2/2/w/1620)

>  [info] 环境变量可以使用的地方  

- URL
- URL params
- Header values
- form-data/url-encoded values
- Raw body content
- Helper fields
- 写 test 测试脚本中
- 通过 postman 的接口，获取或设置环境变量的值。

此处把之前的在 url 中的 IP 地址（或域名）换成环境变量：

![img](https://ask.qcloudimg.com/http-save/yehe-1171488/zfvm8xnp5u.png?imageView2/2/w/1620)

鼠标移动到环境变量上，可以动态显示出具体的值：

![img](https://ask.qcloudimg.com/http-save/yehe-1171488/msdxb4rrwj.png?imageView2/2/w/1620)

再去添加另外一个开发环境：

![img](https://ask.qcloudimg.com/http-save/yehe-1171488/rs4j9obq5m.png?imageView2/2/w/1620)

则可添加完 2 个环境变量，表示两个服务器地址，两个版本：

![img](https://ask.qcloudimg.com/http-save/yehe-1171488/73l36ef55w.png?imageView2/2/w/1620)

然后就可以切换不同服务器环境了：

![img](https://ask.qcloudimg.com/http-save/yehe-1171488/9n52eok8vu.png?imageView2/2/w/1620)

可以看到，同样的变量 server_address，在切换后对应 IP 地址就变成希望的开发环境的 IP 了：

![img](https://ask.qcloudimg.com/http-save/yehe-1171488/h487ru075j.png?imageView2/2/w/1620)

#### [Postman 导出 API 文档中多个环境变量的效果](https://malun666.github.io/aicoder_vip_doc/#/pages/postman?id=postman-%e5%af%bc%e5%87%ba-api-%e6%96%87%e6%a1%a3%e4%b8%ad%e5%a4%9a%e4%b8%aa%e7%8e%af%e5%a2%83%e5%8f%98%e9%87%8f%e7%9a%84%e6%95%88%e6%9e%9c)

顺带也去看看，导出为 API 文档后，带了这种 Environment 的变量的接口，文档长什么样子：

发现是在发布之前，需要选择对应的环境的：

![img](https://ask.qcloudimg.com/http-save/yehe-1171488/dg394n77xg.png?imageView2/2/w/1620)

![img](https://ask.qcloudimg.com/http-save/yehe-1171488/6fqqywrz4h.png?imageView2/2/w/1620)

![img](https://ask.qcloudimg.com/http-save/yehe-1171488/jzcgi6c3lg.png?imageView2/2/w/1620)

发布后的文档，可以看到所选环境和对应服务器的 IP 的：

![img](https://ask.qcloudimg.com/http-save/yehe-1171488/k3tppka6qg.png?imageView2/2/w/1620)

当然发布文档后，也可以实时切换环境：

![img](https://ask.qcloudimg.com/http-save/yehe-1171488/fqnjm0392m.png?imageView2/2/w/1620)

![img](https://ask.qcloudimg.com/http-save/yehe-1171488/6cz9xt9z7y.png?imageView2/2/w/1620)

#### [环境变量的好处](https://malun666.github.io/aicoder_vip_doc/#/pages/postman?id=%e7%8e%af%e5%a2%83%e5%8f%98%e9%87%8f%e7%9a%84%e5%a5%bd%e5%a4%84)

当更换服务器时，直接修改变量的 IP 地址：

![img](https://ask.qcloudimg.com/http-save/yehe-1171488/y6virrs94v.png?imageView2/2/w/1620)

![img](https://ask.qcloudimg.com/http-save/yehe-1171488/udsfrqtqxg.png?imageView2/2/w/1620)

即可实时更新，当鼠标移动到变量上即可看到效果：

![img](https://ask.qcloudimg.com/http-save/yehe-1171488/6i5l61q5gz.png?imageView2/2/w/1620)

### [代码生成工具](https://malun666.github.io/aicoder_vip_doc/#/pages/postman?id=%e4%bb%a3%e7%a0%81%e7%94%9f%e6%88%90%e5%b7%a5%e5%85%b7)

#### [查看当前请求的 HTTP 原始内容](https://malun666.github.io/aicoder_vip_doc/#/pages/postman?id=%e6%9f%a5%e7%9c%8b%e5%bd%93%e5%89%8d%e8%af%b7%e6%b1%82%e7%9a%84-http-%e5%8e%9f%e5%a7%8b%e5%86%85%e5%ae%b9)

对于当前的请求，还可以通过点击 Code

![img](https://ask.qcloudimg.com/http-save/yehe-1171488/fjqiinpdmd.png?imageView2/2/w/1620)

去查看对应的符合 HTTP 协议的原始的内容：

![img](https://ask.qcloudimg.com/http-save/yehe-1171488/ems6iaymhz.png?imageView2/2/w/1620)

#### [各种语言的示例代码Code Generation Tools](https://malun666.github.io/aicoder_vip_doc/#/pages/postman?id=%e5%90%84%e7%a7%8d%e8%af%ad%e8%a8%80%e7%9a%84%e7%a4%ba%e4%be%8b%e4%bb%a3%e7%a0%81code-generation-tools)

比如：

- Swift 语言

![img](https://ask.qcloudimg.com/http-save/yehe-1171488/n2mpfxcdv3.png?imageView2/2/w/1620)

- Java 语言

![img](https://ask.qcloudimg.com/http-save/yehe-1171488/r9dvvipgd1.png?imageView2/2/w/1620)

- 其他各种语言 还支持其他各种语言：

![img](https://ask.qcloudimg.com/http-save/yehe-1171488/n935sl95gc.png?imageView2/2/w/1620)

目前支持的语言有：

- HTTP
- C (LibCurl)
- cURL
- C#(RestSharp)
- Go
- Java 
  - OK HTTP
  - Unirest
- Javascript
- NodeJS
- Objective-C(NSURL)
- OCaml(Cohttp)
- PHP
- Python
- Ruby(NET::Http)
- Shell
- Swift(NSURL)

代码生成工具的好处是：在写调用此 API 的代码时，就可以参考对应代码，甚至拷贝粘贴对应代码，即可。

### [测试接口](https://malun666.github.io/aicoder_vip_doc/#/pages/postman?id=%e6%b5%8b%e8%af%95%e6%8e%a5%e5%8f%a3)

选中某个分组后，点击 Runner

![img](https://ask.qcloudimg.com/http-save/yehe-1171488/p5ijm3jotv.png?imageView2/2/w/1620)

选中某个分组后点击 Run

![img](https://ask.qcloudimg.com/http-save/yehe-1171488/d40ouxqffx.png?imageView2/2/w/1620)

即可看到测试结果： 

![img](https://ask.qcloudimg.com/http-save/yehe-1171488/ynyz14nrsu.png?imageView2/2/w/1620)

关于此功能的介绍可参考[Postman 官网](https://www.getpostman.com/postman)的[git 图](https://www.getpostman.com/img/v2/postman/gifs/collection-runner.gif)

### [MockServer](https://malun666.github.io/aicoder_vip_doc/#/pages/postman?id=mockserver)

直接参考官网。

## [功能界面](https://malun666.github.io/aicoder_vip_doc/#/pages/postman?id=%e5%8a%9f%e8%83%bd%e7%95%8c%e9%9d%a2)

### [多 Tab 分页](https://malun666.github.io/aicoder_vip_doc/#/pages/postman?id=%e5%a4%9a-tab-%e5%88%86%e9%a1%b5)

Postman 支持多 tab 页，于此对比之前有些 API 调试工具就不支持多 Tab 页，比如`Advanced Rest Client`

多 tab 的好处：

方便在一个 tab 中测试，得到结果后，复制粘贴到另外的 tab 中，继续测试其它接口

比如此处 tab1 中，测试了获取验证码接口后，拷贝手机号和验证码，粘贴到 tab2 中，继续测试注册的接口

![img](https://ask.qcloudimg.com/http-save/yehe-1171488/9h475eyqnd.png?imageView2/2/w/1620)

![img](https://ask.qcloudimg.com/http-save/yehe-1171488/f26rmodg14.png?imageView2/2/w/1620)

### [界面查看模式](https://malun666.github.io/aicoder_vip_doc/#/pages/postman?id=%e7%95%8c%e9%9d%a2%e6%9f%a5%e7%9c%8b%e6%a8%a1%e5%bc%8f)

Postman 的默认的 Request 和 Response 是上下布局：

![img](https://ask.qcloudimg.com/http-save/yehe-1171488/ilcrw9qu2a.png?imageView2/2/w/1620)

此处点击右下角的`Two pane view`，就变成左右的了：

![img](https://ask.qcloudimg.com/http-save/yehe-1171488/ij8lqwx97b.png?imageView2/2/w/1620)

>  [info] 左右布局的用途  对于数据量很大，又想要同时看到请求和返回的数据的时候，应该比较有用。  

### [多颜色主题](https://malun666.github.io/aicoder_vip_doc/#/pages/postman?id=%e5%a4%9a%e9%a2%9c%e8%89%b2%e4%b8%bb%e9%a2%98)

Posman 支持两种主题：

- 深色主题

当前是深色主题，效果很不错：

![img](https://ask.qcloudimg.com/http-save/yehe-1171488/jz9xmkcowa.png?imageView2/2/w/1620)

![img](https://ask.qcloudimg.com/http-save/yehe-1171488/u82a7psrgy.png?imageView2/2/w/1620)

- 浅色主题

可以切换到 浅色主题：

![img](https://ask.qcloudimg.com/http-save/yehe-1171488/qu330s3pfl.png?imageView2/2/w/1620)

![img](https://ask.qcloudimg.com/http-save/yehe-1171488/d17b0zdb4c.png?imageView2/2/w/1620)

## [API 文档生成](https://malun666.github.io/aicoder_vip_doc/#/pages/postman?id=api-%e6%96%87%e6%a1%a3%e7%94%9f%e6%88%90)

在服务端后台的开发人员测试好了接口后，打算把接口的各种信息发给使用此 API 的前端的移动端人员时，往往会遇到：

要么是用复制粘贴 -> 格式不友好 要么是用 Postman 中截图 -> 方便看，但是不方便获得 API 接口和字段等文字内容 要么是用 Postman 中导出为 JSON -> json 文件中信息太繁杂，不利于找到所需要的信息 要么是用文档，比如去编写 Markdown 文档 -> 但后续 API 的变更需要实时同步修改文档，也会很麻烦 这都会导致别人查看和使用 API 时很不方便。

-> 对此，Postman 提供了发布 API

预览和发布 API 文档 下面介绍 Postman 中如何预览和发布 API 文档。

### [简要概述步骤](https://malun666.github.io/aicoder_vip_doc/#/pages/postman?id=%e7%ae%80%e8%a6%81%e6%a6%82%e8%bf%b0%e6%ad%a5%e9%aa%a4)

1. Collection
2. 鼠标移动到某个 Collection，点击 三个点
3. Publish Docs
4. Publish
5. 得到 Public URL
6. 别人打开这个 Public URL，即可查看 API 文档

### [预览 API 文档](https://malun666.github.io/aicoder_vip_doc/#/pages/postman?id=%e9%a2%84%e8%a7%88-api-%e6%96%87%e6%a1%a3)

点击分组右边的大于号>

![img](https://ask.qcloudimg.com/http-save/yehe-1171488/s7pgyjn13y.png?imageView2/2/w/1620)

如果只是预览，比如后台开发员自己查看 API 文档的话，可以选择：View in web

![img](https://ask.qcloudimg.com/http-save/yehe-1171488/95ekutmlpe.png?imageView2/2/w/1620)

>  等价于点击Publish Docs去发布：  

![img](https://ask.qcloudimg.com/http-save/yehe-1171488/ca0mw3r3x7.png?imageView2/2/w/1620)

View in Web 后，有 Publish 的选项（见后面的截图）

View in Web 后，会打开预览页面：

比如：

奶牛云

```
https://documenter.getpostman.com/collection/view/669382-42273840-6237-dbae-5455-26b16f45e2b9
```

![img](https://ask.qcloudimg.com/http-save/yehe-1171488/8yfixnbs9s.png?imageView2/2/w/1620)

![img](https://ask.qcloudimg.com/http-save/yehe-1171488/6qtob43od0.png?imageView2/2/w/1620)

而右边的示例代码，也可以从默认的 cURL 换成其他的：

![img](https://ask.qcloudimg.com/http-save/yehe-1171488/gy60gkar15.png?imageView2/2/w/1620)

![img](https://ask.qcloudimg.com/http-save/yehe-1171488/hmptft1z7u.png?imageView2/2/w/1620)

### [发布 API 文档](https://malun666.github.io/aicoder_vip_doc/#/pages/postman?id=%e5%8f%91%e5%b8%83-api-%e6%96%87%e6%a1%a3)

如果想要让其他人能看到这个文档，则点击 Publish：

![img](https://ask.qcloudimg.com/http-save/yehe-1171488/nfsjy8thb8.png?imageView2/2/w/1620)

然后会打开类似于这样的地址：

Postman Documenter

```
https://documenter.getpostman.com/collection/publish?meta=Y29sbGVjdGlvbl9pZD00MjI3Mzg0MC02MjM3LWRiYWUtNTQ1NS0yNmIxNmY0NWUyYjkmb3duZXI9NjY5MzgyJmNvbGxlY3Rpb25fbmFtZT0lRTUlQTUlQjYlRTclODklOUIlRTQlQkElOTE=
```

![img](https://ask.qcloudimg.com/http-save/yehe-1171488/84pbk6ppzq.png?imageView2/2/w/1620)

点击 Publish 后，可以生成对应的公开的网页地址：

![img](https://ask.qcloudimg.com/http-save/yehe-1171488/z3ce5w7dfv.png?imageView2/2/w/1620)

打开 API 接口文档地址：

```
https://documenter.getpostman.com/view/669382/collection/77fd4RM
```

即可看到（和前面预览一样效果的 API 文档了）：

![img](https://ask.qcloudimg.com/http-save/yehe-1171488/84j5ppdtoh.png?imageView2/2/w/1620)

如此，别人即可查看对应的 API 接口文档。

### [已发布的 API 文档支持自动更新](https://malun666.github.io/aicoder_vip_doc/#/pages/postman?id=%e5%b7%b2%e5%8f%91%e5%b8%83%e7%9a%84-api-%e6%96%87%e6%a1%a3%e6%94%af%e6%8c%81%e8%87%aa%e5%8a%a8%e6%9b%b4%e6%96%b0)

后续如果自己的 API 接口修改后：

比如：

![img](https://ask.qcloudimg.com/http-save/yehe-1171488/svcgj7qdll.png?imageView2/2/w/1620)

![img](https://ask.qcloudimg.com/http-save/yehe-1171488/kezpmqdfe8.png?imageView2/2/w/1620)

（后来发现，不用再去进入此预览和发布的流程，去更新文档，而是 Postman 自动支持）

别人去刷新该文档的页面：

```
https://documenter.getpostman.com/view/669382/collection/77fd4RM
```

即可看到更新后的内容：

![img](https://ask.qcloudimg.com/http-save/yehe-1171488/1iumgwfs2k.png?imageView2/2/w/1620)

## [参考资料](https://malun666.github.io/aicoder_vip_doc/#/pages/postman?id=%e5%8f%82%e8%80%83%e8%b5%84%e6%96%99)

- 主要参考：Github: api_tool_postman
- Manage environments
- postman-变量/环境/过滤等 - 简书
- Postman 使用手册 3——环境变量 - 简书
- postman 使用之四：切换环境和设置读取变量 - 乔叶叶 - 博客园

<hr />
