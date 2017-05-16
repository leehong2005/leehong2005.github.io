---
layout:     post
title:      "Mac OS奇技淫巧之 —— 开发技巧使用心得"
subtitle:   " \"Live as if you were to die tomorrow.\" "
date:       2016-10-20 12:00:00
author:     "leehong"
header-img: "img/post-bg-rwd.jpg"
catalog: true
tags:
    - Mac OS
    - Android
    - 工具
---

以下收集了在 Mac OS 上开发 Android 可能经常会使用到的一些命令和技巧，它们可能会极大地提高我们的工作效率。今天把这些奇技淫巧总结在这里，供以后查阅。

---

## 查看 dex, jar 方法数


## 切换 JDK 版本

有时候我们电脑上会安装好几个版本的 JDK，那怎么切换 JDK 的默认版本呢？

可以参考如下文章：[http://blog.csdn.net/tianxiawuzhei/article/details/48263789](http://blog.csdn.net/tianxiawuzhei/article/details/48263789)

#### 如何查看 jdk 的路径

根据苹果的官方说明，Mac OS X 10.5 及以后的版本应该使用 /usr/libexec/java_home 命令来确定 `JAVA_HOME` 

```
//查看默认jdk的安装路径  
/usr/libexec/java_home  
  
//查看jdk 1.6的安装路径  
/usr/libexec/java_home -v 1.6  
```

__注意：__ 这里的 `/usr/libexec/java_home` 是 Mac OS X 10.5 之后才有的，如果直接使用需要注意一下版本限制。

#### 如何设置

1、首先要在 `～/.bash_profile` 配置如下的命令：

```
export JAVA_6_HOME=`/usr/libexec/java_home -v 1.6`
export JAVA_7_HOME=`/usr/libexec/java_home -v 1.7`
export JAVA_8_HOME=`/usr/libexec/java_home -v 1.8`
export JAVA_HOME=$JAVA_6_HOME

# alias命令动态切换JDK版本  
alias jdk6="export JAVA_HOME=$JAVA_6_HOME"  
alias jdk7="export JAVA_HOME=$JAVA_7_HOME"  
alias jdk8="export JAVA_HOME=$JAVA_8_HOME" 
```

2、保存：**source ～/.bash_profile**

3、在终端中执行 `jdk6` 则表示切换到 JDK 1.6 版本，然后你可以再通过 `java -version` 来查看 JDK 的版本了。

> 说明一下：`JAVA_HOME` 最好不要直接把JDK的路径写死了，硬编码容易造成以后的维护不方便。


## 代码统计工具

使用这个命令 **`cloc`**

> 统计代码行数的利器

安装： __brew install cloc__  
用法： __cloc .__

官方文档：[http://cloc.sourceforge.net/](http://cloc.sourceforge.net/)

![](/img/2016/2016-08-09-tools-on-mac-os-cloc.png)

## APK签名

#### 查看keystore信息

使用 `keytool` 命令，详细的可以看 #keytool --help

keytool -list -v -keystore [签名文件] -storepass [密码]
keytool -list -rfc -keystore [签名文件> -storepass [密码]

见下图：

![](/img/2017/2017-05-10-keystore-info.png)


#### apktool

* sudo apktool d [apk文件] —— 得到一个文件夹，以apk名称命名
* sudo apktool b [apk名字] —— 进入文件夹中的dist目录，可以看到一个apk，这个apk文件的签名已经被去掉了

#### jarsigner

* arsigner -verbose -keystore [签名文件keystore] -signedjar [生成的签名文件] [未签名的文件] [keystore的别名]

```java
jarsigner -verbose -keystore demo.keystore -signedjar app-signed.apk app-unsigned.apk demo
```

说明：

-signedjar：它有三个参数，一个是输出的apk，一个是未签名的apk，一个是签名文件keystore的**别名**。

## Gradle

使用 gradle 在编译时动态设置 Android BuildConfig。

**配置字符串类型，如下：**

参考：[http://qiita.com/shts/items/d94834437b22712415c5](http://qiita.com/shts/items/d94834437b22712415c5)

```
gradle.properties
parseApiId=xxxxxxxxxxxxxxxxxxxxx
parseApiKey=xxxxxxxxxxxxxxxxxxxxx
```

```
buildConfigField "String", "PARSE_API_ID", "\"${project.property("parseApiId")}\""
buildConfigField "String", "PARSE_API_KEY", "\"${project.property("parseApiKey")}\""
```

**配置int类型，如下**

buildConfigField "int", "PARSE_API_ID", PARSE_API_ID

## 未完待续

> 其他的一些有用的命令后面再慢慢总结，敬请期待！



（完）



