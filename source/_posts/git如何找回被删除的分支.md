---
title: git如何找回被删除的分支
tags: [git]
copyright: true
date: 2018-11-15 18:37:28
permalink:
categories: git
description: git找回被删除的分支
image: https://static001.geekbang.org/resource/image/86/37/86d40335594d8bfc37e80d0fcbfb9e37.jpg
---
<p class="description"></p>

<img src="https://" alt="" style="width:100%" />

<!-- more -->

在使用git的过程中，因为人为因素造成分支（commit)被删除，可以使用以下步骤进行恢复。

首先用以下步骤创建一个新分支，修改一些文件后删除，以便进行恢复。
1.创建分支 abc

git branch abc
1


2.查看分支列表

git branch -a
  abc
* develop
  remotes/origin-dev/develop
  1
  2
  3
  4


3.切换到abc分支，随便修改一下东西后 commit

切换分支
git checkout abc
Switched to branch 'abc'

创建一个文件
echo 'abc' > test.txt

commit
git add .
git commit -m 'add test.txt'
[abc 3eac14d] add test.txt
 1 file changed, 1 insertion(+)
 create mode 100644 test.txt
1
2
3
4
5
6
7
8
9
10
11
12
13


4.删除分支abc

git branch -D abc
Deleted branch abc (was 3eac14d).
1
2


5.查看分支列表，abc分支已不存在

git branch -a
* develop
  remotes/origin-dev/develop
  1
  2
  3


恢复步骤如下：
1.使用git log -g 找回之前提交的commit
commit 3eac14d05bc1264cda54a7c21f04c3892f32406a
Reflog: HEAD@{1} (fdipzone <fdipzone@sina.com>)
Reflog message: commit: add test.txt
Author: fdipzone <fdipzone@sina.com>
Date:   Sun Jan 31 22:26:33 2016 +0800

    add test.txt

1
2
3
4
5
6
7
8
9


2.使用git branch recover_branch[新分支] commit_id命令用这个commit创建一个分支
git branch recover_branch_abc 3eac14d05bc1264cda54a7c21f04c3892f32406a

git branch -a
* develop
  recover_branch_abc
  remotes/origin-dev/develop
  1
  2
  3
  4
  5
  6
  可以见到recover_branch_abc已创建 


3.切换到recover_branch_abc分支，检查文件是否存在
git checkout recover_branch_abc
Switched to branch 'recover_branch_abc'

ls -lt
total 8
-rw-r--r--   1 fdipzone  staff     4  1 31 22:38 test.txt
1
2
3
4
5
6
这样就可以恢复被误删的分支了
--------------------- 


原文：https://blog.csdn.net/fdipzone/article/details/50616386 
版权声明：本文为博主原创文章，转载请附上博文链接！

<hr />
