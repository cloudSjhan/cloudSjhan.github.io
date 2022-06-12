---
title: git常用操作
tags: [git]
copyright: true
date: 2018-12-02 10:38:11
permalink:
categories: git
description: git常用基本操作
image: https://static001.geekbang.org/resource/image/39/f4/392d92326d26edef2b1689c236f3d5f4.jpg
---
<p class="description"></p>

<img src="https://" alt="" style="width:100%" />

<!-- more -->

首先来一遍从fork到pull request这个过程的基础流程
首先，fork 一个repository，实际上是复制了一份 repository 到自己的 GitHub 账户下，然后就可以从 GitHub 将它 clone 到你的电脑上，命令如下：

git clone <URLFROMGITHUB>
连接到原始的Repository，因为如果原始的Repository内容有所改变时，我们希望能够pull这些变化，所以新增一个远端链接，并把它命名为’upstream’，命令如下：

git remote add upstream <URLFROMUPSTREAMGITHUB>
新增branch分支，并选用新增分支。避免与主分支master造成冲突，当我们在新增分支上完成了自己的功能后再合并到主分支，命令如下：

git branch <BRANCHNAME>
git checkout <BRANCHNAME>
git checkout -b <BRANCHNAME> –创建新的分支并切换到新的分支上
记录，在我们自己的分支上修改后，需要记录下来。

git status –查看当前状态
git add -A –记录修改文件，加上 -A，會將新增檔案跟刪除檔案的動作一起記錄下來
git commit -m "add a file" –提交全部修改
git checkout master –第二天开始工作前，切换到master分支
git pull origin master –从master的远程分支拉取代码
git checkout <BRANCHNAME> –切换到task所在的本地分支
git rebase -i master –将master上的最新的代码合并到当前分支上，这里的-i的作用是将我们 当前分支之前的commit压缩成为一个commit，这样做的好处在于当我们之后创建pull request并进行相应的code review的时候，代码的改动会集中在一个commit，使得code review更直观方便
git push --set-upstream origin <my branch name> –最后，当task的所有编码完成之后，将代码push到远程分支
先获取远端，再提交，每次提交代码前，都需要先获取最新代码，防止覆盖他人代码

git fetch --dry-run –检查远端是否有变动
git pull –从远端分支更新最新代码
建立Pull Requests，进入你的github项目页，一般情况下 GitHub会检测到你有了新的推送，会主动提示你，点击Create pull request，写上说明，再按Send pull request就完成了，如果 Pull Request 沒有问题的话，很快就會被自动合并 merged 了哦！

本地合并分支，并删除分支，将分支合并到主分支上，并删除之

git checkout master –首先切换到主分支中
git merge <BRANCHNAME> –合并另一个分支进来
git branch -d <BRANCHNAME> –删掉刚刚合并的分支
git push <REMOTENAME> --delete <BRANCHNAME> –也可以把合并分支从GitHub上的副本repository中刪除
其他常用命令
git init –将一个文件夹初始化为git仓库
git status –检查当前repository中的修改
git diff –查看对文件的修改
git add <FILENAME> –准备提交对于一个文件的修改
git add . –准备提交对所有文件的修改
git commit -m "<your commit message>" –提交你所准备好的修改，并附上简短说明
git config --global user.username <USerNamE> –配置github账号
git remote add <REMOTENAME> –新增远端链接
git remote set-url <REMOTENAME> –对一个远端设定地址
git remote add <REMOTENAME> <URL> –新增带地址的远端链接
git remote -v –查看所有远端
git pull <REMOTENAME> <BRANCHNAME> –从一个远端收取更新（默认为主分支）
git push <REMOTENAME> <BRANCHNAME> –提交代码到指定远端（默认为主分支）
git branch -M <NEWBRANCHNAME> –修改当前分支名字
git branch –列出所有分支

<hr />
