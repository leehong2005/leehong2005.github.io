---
layout:     post
title:      "Android Studio多Module使用aar编译报错的解决方案"
subtitle:   " \"What doesn't kill you makes you stronger.\" "
date:       2016-08-28 00:00:00
author:     "leehong"
header-img: "img/post-bg-universe.jpg"
catalog: true
tags:
    - Android
    - AAR
    - Android Stuido
---

> * AAR Format 
* 在Gradle中如何使用aar
* 多Module中使用aar编译报错
* 解决方案

## AAR Format

在 `Android Studio` 之前，如果用引用第三方的库，一般使用 `jar` 包，它只包含了class，没有包含对应的资源、so库等，所以引用起来就不方便，特别是一些 UI 库，第三方在使用的时候，还需要自己单独导入对应的资源（字符串、图片等）。

现在 Android 中引入了 `aar` 这种包结构，它其实也是一个zip包，它里面包含了很多资源，具体的结构如下所示：

参考链接：[AAR Format](http://tools.android.com/tech-docs/new-build-system/aar-format)

---

The 'aar' bundle is the binary distribution of an Android Library Project.

The file extension is .aar, and the maven artifact type should be aar as well, but the file itself a simple zip file with the following entries:

* /AndroidManifest.xml (mandatory)
* /classes.jar (mandatory)
* /res/ (mandatory)
* /R.txt (mandatory)
* /assets/ (optional)
* /libs/*.jar (optional)
* /jni/<abi>/*.so (optional)
* /proguard.txt (optional)
* /lint.jar (optional)

These entries are directly at the root of the zip file. 

The R.txt file is the output of aapt with --output-text-symbols.

---

## 在Gradle中如何使用aar

Android Studio (AS) 中使用aar，通常是把aar文件放到 `libs` 目录下面，需要在 __MODULE__ 目录下面 `build.gralde` 中添加如下配置：

```java
repositories {
    flatDir {
        dirs 'libs'
    }
}
```

另外，还需要在 `dependencies` block中添加如下代码：

```java
dependencies {
	compile(name: 'aar-file-name', ext: 'aar')  
	// 其中aar-file-name不用后缀名
}
```

## 多Module中使用aar编译报错

在开发Android App的时候，如果项目比较复杂，都可能会有很多个 `library` 工程，考虑如下场景：

* Module A：Library
* Module B：Library
* Module C：App

其中 `Module B` 依赖 `Module A`，`Module C` 依赖 `Module A`，`Module B`。

如果在 Module A 中添加了aar库的话，则编译的时候，可能会报错，_例如在 Module A 中使用了gif库，_ 对应的引用代码是：

```java
compile(name: 'gif-1.0', ext: 'aar')
```

在编译的时候，错误信息可能如下所示：

```
Failed to resolve: gif-1.0
```

Module A 是可以正常编译通过的，但是编译 Module C 的时候，就会发现找不到对应的 gif 库，因为在 Module A 中指定的 libs 目录是相对的，在 Module C 中的 `build.gradle` 只会从自己所在目录下面的 libs 中查找，结果是找不到 gif 相关的 aar 库，所以就会编译出错。


## 解决方案

上述编译错误的根本原因是不 Module C 在编译的时候找不到对应的 gif 库，所以，根本要解决的就是如何让所有的工程都能引用到呢？

#### 方案一

所有依赖 Module A 的 Module 都添加：

```java
repositories {
    flatDir {
        dirs 'xxx/libs' // Module A的libs的目录地址
    }
}
```

把所有的 Module 都添加上 Module A 的 libs 目录的相对地址。
 
这种方案不也的地方在于：__这样改动太多，而且如果以后多增加一个 Module 的话，也必须记得添加这行，否则会编译不过，这个方案不好，不具备扩展性。__

#### 方案二

有没有一种更好的方案呢？在网上查了一些，从一些方案中想到了最终的方案，本质上是要保证所有的 Module 都能引用到某个目录下面的 aar 库不就行了吗？基于这个思路去想，就想到能否在 `Project` 下的 `build.gradle` 中的 `repositories` 中添加相应的引用呢？默认的配置如下：

```java
allprojects {
    repositories {
        jcenter()
    }
}
```

那就试一试吧，在 `repositories` 下添加 `flatDir` 试试。代码如下：

__注意：这里一定是在 Project 下面的 build.gralde 中添加。__

```java
dirs project(':AppLibrary').file('libs')
```

编译，通过了，耶～～

__最终代码是这样的：__

```java
allprojects {
    repositories {
        jcenter()

        flatDir {
            // 由于Library module中引用了 gif 库的 aar，在多 module 的情况下，
            // 其他的module编译会报错，所以需要在所有工程的repositories
            // 下把Library module中的libs目录添加到依赖关系中
            dirs project(':AppLibrary').file('libs')  
        }
    }
}
```

大家也可以参考这个链接，我从这上面也得到了一些想法和帮助。


[Adding local .aar files to Gradle build using “flatDirs” is not working](http://stackoverflow.com/questions/24506648/adding-local-aar-files-to-gradle-build-using-flatdirs-is-not-working)


__总的说来，方案二比较完美地解决了多 module 引用 aar 库的问题，推荐使用这种方法。__


（完）