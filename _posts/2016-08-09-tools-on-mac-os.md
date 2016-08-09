---
layout:     post
title:      "Mac OS奇技淫巧之 —— 小工具使用心得"
subtitle:   " \"Some tools on Mac OS!\" "
date:       2016-08-09 12:00:00
author:     "leehong"
header-img: "img/post-bg-rwd.jpg"
catalog: true
tags:
    - Mac OS
    - 工具
    - 命令行
---

使用 mac OS 也有一段时间了，在这期间，学习了一些命令行，这些命令能极大地提高我们的工作效率。今天把这些奇技淫巧总结在这里，供以后查阅。

---

## Homebrew

官方网址在这里：[Homebrew](http://brew.sh/index_zh-cn.html)  
这个软件基本是必备，很多命令都可以直接通过它来安装，像 `git`, `svn`等，它类似于 `apt-get` 命令，很多软件直接通过这个命令就可以安装。例如要安装 `svn` ，命令行如下：

```java
brew install svn
```

要卸载的安装的软件，命令行如下：

```java
brew uninstall svn
```

## ag

`ag` 命令是从当前路径下的文件中查找指定的字符串，简单的说就是内容查找。通常在工程中看到某个字符串（类名，变量等字符串）是否在别的有使用，直接使用这个命令能非常快速的得到结果。

> **速度很快，这才是重点。**

#### 如何安装

基于上文提到的 `Homebrew` 来安装：`brew install ag` ，安装过程如下图所示：
![安装 ag 命令](/img/2016/2016-08-09-tools-on-mac-os-ag-install.png)
安装好后，可以使用 `ag --version` 和 `ag --help` 来查看版本和帮助信息。

#### 如何使用

例如我要在某个路径下面查找字符串 *WebView*，直接这样： `ag 'WebView'`，检查结果如下图所示：
![搜索结果图](/img/2016/2016-08-09-tools-on-mac-os-ag-use.png)


## zsh

## fzf

## tree

`tree` 是以树状结构显示当前文件或文件夹的命令，直接通过 `Homebrew` 安装即可。
可以检查帮助来了解详细的使用方法，一般常用的：`tree -L 2` ，表示最大显示两层，运行结果如下：
![结果图](/img/2016/2016-08-09-tools-on-mac-os-tree-use.png)

## Go2Shell

想想这些场景：

* 有时我们要在 terminal 中进入某个路径很长的文件夹下，要一层一层地通过命令来打，这得输多少个 cd 命令呀，效率比较慢。
* 要知道 Finder 中的一个文件夹的路径是什么？mac上还没啥好办法？至少我现在不知道～～

发现了一个比较好的工具 `Go2Shell` ，从 AppStore 上就可以下载安装。它可以配合 Finder 一起来使用。
在 Finder 中打 Application 文件夹，按住 command 键，拖动 Go2Shell 的图标到 Finder 菜单就可以在 Finder 快捷打开Go2Shell了。

![](/img/2016/2016-08-09-tools-on-mac-os-go2shell-use.png)

在 Finder 中点击这个图标，就可以直接打开 terminal 并且定位到当前目录 —— 场景一就解决了，哈哈！！  
打开 terminal 后，直接输入 `pwd` ，就可以得到当前的路径了 —— 场景二就解决了，哈哈！！





