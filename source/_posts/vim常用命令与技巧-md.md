---
title: vim常用命令与技巧(不定期更新).md
tags: [vim]
copyright: true
date: 2019-04-21 21:38:14
permalink:
categories: vim
description: vim常用的命令与技巧总结
image: https://static001.geekbang.org/resource/image/da/89/dad54cea1adc9afb20600c916931be89.jpg
---
<p class="description"></p>

<img src="https://" alt="" style="width:100%" />

<!-- more -->

## Vim常用的命令与技巧总结：

1. 在每行行首添加相同的内容：

```shell
:%s/^/要添加的内容
```



2. 在每行行尾添加相同的内容：

```shell
:%s/$/要添加的内容
```



3. 利用正则表达式删除代码段每行的行号

```shell
:%s/^\s*[0-9]*\s*//gc

其中，^表示行首，$表示行尾，\s表示空格，[0-9]表示0~9的数字，*表示0或多个，%s/^\s*[0-9]*\s*//gc的意思是将每行以0或多个空格开始中间包含0或多个数字并以0或多个空格结束的字符串替换为空。
```

4. 指定行首添加"#"

```shell
:447,945 s/^/#
447-945行的行首添加 #
```

5. 删除每行前面的内容

```shell
:10,15 s/^/#//gc
```

6. 统计m到n行中"字符串"出现的次数

```shell
:m,n s/字符串//gn
```

7. 统计"字符串"在当前编辑文件出现的次数

   ```shell
   : %s/字符串/ng
   ```

8. 统计词语在文件中出现的行数:

```shell
cat file|grep -i 字符串 |wc -l
```

9. pycharm中vim插件批量缩进：

```shell
:m,n >
//向右缩进4空格

:m,n < 
//向左缩进4空格
```

10. 跳转到行首: ^
11. 跳转到行尾:$
12. 跳转到文件开头: gg
13. 跳转到行尾：G



<hr />
