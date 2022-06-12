---
title: git-保持fork的项目与上游同步
tags: [git]
copyright: true
date: 2019-02-26 09:48:38
permalink:
categories: git
description: git-保持fork的项目与上游同步
image: https://ws1.sinaimg.cn/large/006tKfTcly1g0jl7ibkuwj30xc0hidi6.jpg
---
<p class="description"></p>

<img src="https://" alt="" style="width:100%" />

<!-- more -->

- 添加上游仓库：

git remote add upstream [upstream_url]


- fetch 之：

git fetch upstream


- 切换到本地master分支：

git checkout master


- 将upstream/master merge到 本地master分支：

git merge upstream/master  //可能会报错，如果报错就执行：git pull


- 同时push到自己的github仓库：

git push origin master


<hr />
