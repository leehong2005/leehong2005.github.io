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

---


## Go2Shell

想想这些场景：

* 有时我们要在 terminal 中进入某个路径很长的文件夹下，要一层一层地通过命令来打，这得输多少个 cd 命令呀，效率比较慢。
* 要知道 Finder 中的一个文件夹的路径是什么？mac上还没啥好办法？至少我现在不知道～～

发现了一个比较好的工具 `Go2Shell` ，从 AppStore 上就可以下载安装。它可以配合 Finder 一起来使用。

__在 Finder 中打 Application 文件夹，按住 command 键，拖动 Go2Shell 的图标到 Finder 菜单就可以在 Finder 快捷打开Go2Shell了。__

![](/img/2016/2016-08-09-tools-on-mac-os-go2shell-use.png)

* __在 Finder 中点击这个图标，就可以直接打开 terminal 并且定位到当前目录 —— 场景一就解决了，哈哈！！__
* __打开 terminal 后，直接输入 `pwd` ，就可以得到当前的路径了 —— 场景二就解决了，哈哈！！__


---

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

---

## zsh

![](/img/2016/2016-08-09-tools-on-mac-os-zsh.png)

__官方网站：[点这里](http://ohmyz.sh/)__

zsh的安装：

```shel
$ sh -c "$(curl -fsSL https://raw.github.com/robbyrussell/oh-my-zsh/master/tools/install.sh)"
```

Shell是Linux/Unix的一个外壳，你理解成衣服也行。它负责外界与Linux内核的交互，接收用户或其他应用程序的命令，然后把这些命令转化成内核能理解的语言，传给内核，内核是真正干活的，干完之后再把结果返回用户或应用程序。

Linux/Unix提供了很多种Shell，为毛要这么多Shell？难道用来炒着吃么？那我问你，你同类型的衣服怎么有那么多件？花色，质地还不一样。写程序比买衣服复杂多了，而且程序员往往负责把复杂的事情搞简单，简单的事情搞复杂。牛程序员看到不爽的Shell，就会自己重新写一套，慢慢形成了一些标准，常用的Shell有这么几种，sh、bash、csh等，想知道你的系统有几种shell，可以通过以下命令查看：

```java
cat /etc/shells
```

显示如下：

```java
/bin/bash
/bin/csh
/bin/ksh
/bin/sh
/bin/tcsh
/bin/zsh
```

这就是当前系统支持的 shell 系统。

通过下面命令可以切换 shell 系统：

```java
/usr/bin/chsh -s /bin/bash  // 让终端重置为bash
/usr/bin/chsh -s /bin/zsh   // 让终端重置为zsh
```

在 `.zshrc` 中添加：  
[ -f ~/.fzf.zsh ] && source ~/.fzf.zsh  
[ -f ~/.fzf.zsh ] && source ~/.bash_profile

> 注意：  
有时可能会遇到这样的问题： 
例如我要通过 git 添加文件，这样用 `git add *.java`，这样可能会出现 __zsh:no matches found: *.java__ 之类的错误，需要这样解决：__在.zshrc中添加setopt no_nomatch__，原因是这个命令如果在zsh中的话，zsh会优先去角析 * ，而不会把这个命令交到 git add 去处理。

参考链接：[点我](http://glanwang.com/2016/05/31/Mac/zsh%E4%B8%8D%E5%85%BC%E5%AE%B9%E7%9A%84%E5%9D%91/)

---

## fzf

TBD



---

## tree

`tree` 是以树状结构显示当前文件或文件夹的命令，直接通过 `Homebrew` 安装即可。
可以检查帮助来了解详细的使用方法，一般常用的：`tree -L 2` ，表示最大显示两层，运行结果如下：
![结果图](/img/2016/2016-08-09-tools-on-mac-os-tree-use.png)

---

## cloc

> 统计代码行数的利器

安装： __brew install cloc__  
用法： __cloc .__

![](/img/2016/2016-08-09-tools-on-mac-os-cloc.png)

___


## findreplace.sh

> __这个脚本的功能在指定的文件夹下，从文件中查找指定字符串，替换成目标字符串。__ 

有的时候我们需要对某些文件中的文本进行全局替换，这个脚本是基于 `sed` 命令来实现，也是从网上参考了很多。

#### 参考链接

* [Recursive search and replace in text files on Mac and Linux](http://stackoverflow.com/questions/9704020/recursive-search-and-replace-in-text-files-on-mac-and-linux)

* [https://gist.github.com/nateflink/9056302](https://gist.github.com/nateflink/9056302)

#### 实现如下

```
#!/bin/bash
#By Nate Flink

#Invoke on the terminal like this
#curl -s https://gist.github.com/nateflink/9056302/raw/findreplaceosx.sh | bash -s "find-a-url.com" "replace-a-url.com"
 
if [ -z "$1" ] || [ -z "$2" ]; then
  echo "Usage: ./$0 [find string] [replace string]"
  #exit 1
fi
 
FIND=$1
REPLACE=$2

#needed for byte sequence error in ascii to utf conversion on OSX
export LC_CTYPE=C;
export LANG=C;
 
#sed -i "" is needed by the osx version of sed (instead of sed -i)
find . -type f -exec sed -i "" "s/${FIND}/${REPLACE}/g" {} +
#exit 0
```


#### 用法

上面的代码保存到 `findreplace.sh` 文件中，然后把它放到 `/usr/local/bin/` 目录下面，可以全局访问，放好后，在终端中输入 `which findreplace.sh` ，如果没有问题，会有输出 `/usr/local/bin/findreplace.sh`。

用的时候就这样，在终端中输入：

```
. findreplace "source" "target"
```



---


## 未完待续

> 其他的一些有用的命令后面再慢慢总结，敬请期待！



（完）



