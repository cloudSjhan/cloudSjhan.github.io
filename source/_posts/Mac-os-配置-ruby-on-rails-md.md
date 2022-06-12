---

title: Mac os 环境配置ruby on rails 及其Hello world
tags: [ruby on rails, web]
copyright: true
date: 2018-08-26 23:20:33
permalink:
categories: ruby
description:
image:
---
<p class="description"></p>

<!-- more -->

今天在Mac OS环境中倒腾ruby on rails，遇到一些坑并排坑后总结一个搭建过程，供大家参考。

### 大纲

- 本着IT届能用最新的就不用前面的版本的宗旨，在进行之前必须将你的Mac升级到最新的macOS High Sierra

- 安装 XCode Command Line Tools

- 配置Git

- 安装Homebrew

- 安装GPG

- 安装RVM

- 安装ruby

- 升级RubyGems

- 安装rails

- 基本MVC探究之Hello world

  ### Ruby On rails for mac os High Sierra

  * Mac OS是自带ruby的，但是这些ruby的版本都不是最新的，我们也不要用这些过时的版本

  * 首先，升级你的Mac OS到10.13

  * 查看是否安装xcode command line tool：

    ```shell
    $:xcode-select -p
    如果你看到：
    xcode-select: error: unable to get active developer directory...
    说明你没有安装xcode command line tool,需要按照下面的步骤安装。
    ```

    ```shell
    如果你看到：
    $:/Applications/Xcode.app/Contents/Developer 或者/Library/Developer/CommandLineTools
    恭喜你，xcode command line tool你已经安装好了
    ```

    ```
    But，如果你很不幸运地看到了这句话：
    $: /Applications/Apple Dev Tools/Xcode.app/Contents/Developer
    那么你就要卸掉xcode重新安装了，具体原因看
    ```

    [这里](http://rvm.io/support/faq#can-i-use-a-path-with-spaces)

  * 安装xcode

  * ```shell
     xcode-select --install
    ```

  * 一路确认之后，就可以安好xcode，但是如果你的网速不好，等待时间过长，你可以从[这里](https://developer.apple.com/downloads/more)输入你的APPID下载。

  * 确认一下是否安好

  * ```shell
    $ xcode-select -p
    /Library/Developer/CommandLineTools
    ```

  ### 配置Git

  - 在安装ruby on rails 之前，你应该配置你的Git。Git在Mac OS上使自动安装的软件

  - 检查Git版本并确认已经安装让你放心

  - ```shell
    $ git version
    git version 2.4.9 (Apple Git-60)
    ```

  - 配置Git之前，你应该到[GitHub](https://help.github.com/articles/signing-up-for-a-new-github-account/)上注册你的账号并记住密码和邮箱。并使用下面的命令配置：

    ```shell
    $ git config -l --global
    fatal: unable to read config file '/Users/.../.gitconfig': No such file or directory
    $ git config --global user.name "Your Real Name"
    $ git config --global user.email me@example.com
    $ git config -l --global
    user.name=Your Real Name
    user.email=me@example.com
    ```

  - Git配置完成，在你想用Git的时候，它就会蹦出来了。

  ### 安装Homebrow

  * 检查homebrow是否已经安装

    ```shell
    $ brew
    -bash: brew: command not found
    ```

    RVM需要[Homebrow](http://brew.sh/),其实一个Mac OS额安装包管理工具，用来下载一些软件，类似于Ubuntu的apt-get和centos的yum install.为避免安装RM出现问题，我们必须安装homebrow：

    ```shell
    $ ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
    ```

    安装过程中可能会出现一些warning并让你输入密码：

    ```shell
    WARNING: Improper use of the sudo command could lead to data loss...
    To proceed, enter your password...
    Password:
    ```

    尽管输入密码，忽略warning。

    我们这里是使用了Mac OS内置的ruby来安装homebrow。

  ### 安装GPG

  * [gpg](https://en.wikipedia.org/wiki/GNU_Privacy_Guard)是一个用来检查RVM下载包的安全性的程序，我们使用homebrew来安装gpg:

  * ```shell
    $ brew install gpg
    ```

  * gpg安装之后，为RVM安装key:

    ```shell
    $ command curl -sSL https://rvm.io/mpapis.asc | gpg --import -
    ```

  ### 安装RVM

  * [RVM](https://rvm.io/)，是Ruby version manager的简写，用来安装ruby或者管理rails版本。[这个网站](https://rvm.io/rvm/install/)详细说明了安装ruby的方式，但是我们有一种最简便的方式：

  * ```shell
    $ \curl -L https://get.rvm.io | bash -s stable
    ```

    "curl"前面的“\”用来避免ruby版本的冲突，不要漏掉。

  * 安装过程中你可能会看到

    ```shell
    mkdir: /etc/openssl: Permission denied
    mkdir -p "/etc/openssl" failed, retrying with sudo
    your password required for 'mkdir -p /etc/openssl':
    ```

    请输入密码并继续。

  * 如果你已经安装过RVM，使用下面的命令update：

    ```shell
    $ rvm get stable --autolibs=enable
    ```

  * 重启terminal窗口或者使用：使RVM生效

    ```shell
    $ source ~/.rvm/scripts/rvm
    ```

  ### 安装ruby

  * 在安装RVM之后，我们安装最新版本的ruby。ruby 2.5.1是写此博客时当前最新的ruby版本，还请查看ruby[官网]($ source ~/.rvm/scripts/rvm)查看最新版本的ruby。必须指定ruby的版本：

    ```shell
    $ rvm install ruby-2.5.1
    ```

    安装后检查是否安装成功：

    ```shell
    $ ruby -v
    ruby 2.5.1...
    ```

  ### 升级rubyGemset

  * [RubyGems](https://rubygems.org/gems/rubygems-update)是一个ruby的包管理工具，用来安装ruby的工具或者额外功能的包。

  * 查看gem版本：

    ```shell
    $ gem -v
    ```

    将gem升级到最新版本

    ```shell
    $ gem update --system
    ```

  * 显示RVM gemsets的最初两个设置

    ```shell
    $ rvm gemset list
    gemsets for ruby-2.5.0
    => (default)
       global
    ```

    一般使用global：

    ```shell
    $ rvm gemset use global
    ```

  * 安装bundle,[Bundle](https://rubygems.org/gems/bundler)是一个管理gem的必须的工具

    ```shell
     $ gem install Bundler
    ```

  * 安装Nokogiri，[Nokogiri](http://nokogiri.org/)需要编译成指定的系统，在上面的配置下，号称最难安装的包，也将安装好

    ```
    $ gem install nokogiri
    ```

    如果你真的不幸运在安装时遇到问题，Stack [Overflow](http://stackoverflow.com/questions/tagged/nokogiri)能帮到你。

  ### 安装rails

  * [这里](http://rubygems.org/gems/rails)是ruby On rail最新的版本，5.1是最新稳定版本，5.2是release版本，我们安装5.1.

    ```shell
    $ gem install rails --version=5.1
    ```

    如果你喜欢尝鲜，可以使用

    ```shell
    $ gem install rails --pre
    ```

    安装release版本。

    检查一下rails是否装好：

    ```shell
    $ rails -v
    Rails 5.2.0
    ```

  * 到此为止，ruby on rails 以及其环境配置都已妥当，可以开始你的ruby之旅了。

  ### ruby on rails 的Hello world

  * ```shell
    $ cd /
    $ mkdir worlspace
    $ cd workspace
    $ rails _5.1.0_ new hello_app
    $ cd hello_app
    $ rails server
    ```

    将http://localhost:3000输入浏览器，就能看到ruby on rails的欢迎界面。

