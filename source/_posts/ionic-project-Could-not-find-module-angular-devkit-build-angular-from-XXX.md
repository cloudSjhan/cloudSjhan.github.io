---
title: ionic project-Could not find module @angular-devkit/build-angular from XXX
tags: [angular,ionic]
copyright: true
date: 2018-11-30 22:02:49
permalink:
categories: frontEnd
description: ionic project-Could not find module @angular-devkit/build-angular from XXX
image: https://static001.geekbang.org/resource/image/58/ff/587a987ab448398004cee9282d0984ff.jpg
---
<p class="description"></p>

<img src="https://" alt="" style="width:100%" />

<!-- more -->

- 从GitHub上找了一个ionic的demo，准备运行一下，报错：Could not find module @angular-devkit/build-angular from XXX

- 操作步骤：

  - git clone https://github.com/ionic-team/ionic-conference-app.git

  - ionic serve

  - 报错：

    ```
     Could not find module "@angular-devkit/build-angular" from
    
    ```

    解决方案：

    -  npm install --save @angular-devkit/build-angular


<hr />



