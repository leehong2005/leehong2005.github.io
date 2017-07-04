---
layout:     post
title:      "Android Lottie动画库研究"
subtitle:   " \"Discontent is the first step in progress.\" "
date:       2017-06-30 12:00:00
author:     "leehong"
header-img: "img/post-bg-android.jpg"
catalog: true
tags:
    - Android
    - Animation
    - Lottie
---


随着移动互联网的发展，越来越多的移动 APP 都会从交互视觉方面来提升用户体验，其中提供炫酷的动画效果是一个经常使用的方法。然而，众所周知，Android平台中的动画效果的实现一直以来都存在一些痛点，而这些痛点也给 Android平台应用上实现丰富动画效果带来了非常大的困难。本文将会提及一种更先进的动画框架，通过这个框架可以方便实现各种炫酷的动画效果，而且相比传统动画效果实现方法，有非常大的优势，各位看官，不要着急，请接着慢慢往下看！

### 一. 传统动效实现

传统动画效果的实现方法，归纳总结起来，大概有这么几种方案：

##### 1. 代码实现

通过 Android 系统提供的动画接口，通过代码来实现动画效果，一般情况下，这种动画效果都是比较简单的，例如实现一个简单的平移、绽放，旋转等，如果这三种效果组合起来，将会变得非常复杂，因此，代码实现一般也仅适用于简单动效的开发，如果太复杂的动效的话，那么投入产出比将会严重不平衡。

通过代码实现，也又分两种：

* 纯代码实现：使用 Animation 类或 Property Animation 来实现。
* 序列帧实现：设计师提供一系列的序列帧图片，开发者将这些图片封装成 xml animation，然后在代码中加载该 xml，从而实现动画效果。

这种方式的实现的缺点：

* 不灵活：代码实现复杂，动画效果的更改，将更改代码。
* 开发效率低下：需要多次调试和修改，才能达到最终的实现效果。
* 内存开销大：序列帧是多张图片实现，内存开销大。

##### 2. Gif 动态图

通过 Gif 图片可以实现动效，但是最大的问题是：

* 内存大
* 不能控制动画，例如暂停，开始

##### 3. 小视频

播放一个轻量的小视频，这种也能达到效果，但最大的问题是：

* 兼容性，视频播放特殊性可能引发兼容的问题
* View 不好控制，可能会影响到音频等


总结起来，传统动效实现的流程如下：

**视觉出设计稿 ——> 输出 n 张图 ——> 研发实现序列帧 ——> 运行看效果/适配 ——> 调整代码再看效果 ——> ...**

所下图所示：

![](/img/2017/2017-06-30-lottie-01.png)

由此可以看出，开发者实现一个动画非常繁琐复杂，而且每增加一种动画，需要重新开发。

### 二. 传统动效实现的缺点

综合看来，以上几种方案都有非常明显的缺点，再次概括一下，传统动画效果的缺点有

##### 1. 效率低

设计师要出多终序列帧图片，工作量大；研发需要针对这些序列帧来实现，反复调试，开发工作量大，从而导致一个动画效果的实现的整体效率非常低。

##### 2. 不灵活

要更新一个动画，需要研发再次修改代码，再反复调试，非常不灵活。

那针对这些问题，有没有一种能全新的解决方案来解决这些问题，使动画效果的实现更加优雅更高效呢？接下来就是本文的重点 —— `Airbnb Lottie` 动画库登场！

### 三. Airbnb Lottie动画框架的实

关于 `lottie`（[https://github.com/airbnb/lottie-android](https://github.com/airbnb/lottie-android) ）， 它提供了优雅的解决方案，通过这个框架，可以做出非常丰富的炫酷的动画效果。先来看一下官方的效果图：

![](/img/2017/2017-06-30-lottie-02.gif)

![](/img/2017/2017-06-30-lottie-03.gif)

![](/img/2017/2017-06-30-lottie-04.gif)

![](/img/2017/2017-06-30-lottie-05.gif)

#### 1. 基于 lottie 动效实现流程

* **第一步：** 设计师设计动画效果，再通过插件（[Adobe After Effects](http://www.adobe.com/hk_zh/products/aftereffects.html)）将动画效果导出成一个动画描述文件
* **第二步：** 将这个动画描述文件，预置在应用的 `assets` 目录下，通过`lottie`框架加载这些动画文件。
* **第三步：** 运行，查看效果

概括起来，如下图所示：

![](/img/2017/2017-06-30-lottie-06.png)

#### 2. Lottie的优势

相比传统的实现方案，具备非常大的优势。

##### a) 高效

开发者一次开发，可以多次复用，不需要再去写各种具体的动画相关的代码。设计师设计好动画效果之后，导出文件即可。

##### b) 灵活

由于动画通过文件来描述，替换不同的文件，将会得到不同的动画效果，动效的更新或升级，将非常灵活。

##### c) 从网络加载

既然是加载动画描述文件，那么这个文件就可以从任意地方来，assets、sdcard、network都是可以的。从网络加载动画描述文件，将能做到不发版的情况下，动态更新动效。

##### d) 多平台支持

动画文件可以应用于 Android 和 iOS 平台，这样设计师只需出一份动效设计稿就行，不用区分平台。

**不同的动画效果，只需要做一次开发即可**

### 四. Lottie实现原理

* 框架图

![](/img/2017/2017-06-30-lottie-07.png)

* 实现原理

Lottie使用json文件来作为动画数据源，json文件是通过Bodymovin插件导出的，查看sample中给出的json文件，其实就是把图片中的元素进行来拆分，并且描述每个元素的动画执行路径和执行时间。Lottie的功能就是读取这些数据，然后绘制到屏幕上。

首先要解析json，建立数据到对象的映射，然后根据数据对象创建合适的Drawable绘制到view上，动画的实现可以通过操作读取到的元素完成。

具体过程如下所示：

>json文件 ——> Componsition ——> Drawable ——> View

通过如下3个核心类来来完成整个工作流程，因而使用起来比较简单。

**LottieComposition (json->数据对象)**
	
Lottie使用LottieComposition来作为After Effects的数据对象，即把Json文件映射为到LottieComposition，该类中提供了解析json的静态方法。

**LottieDrawable (数据对象->Drawable)**

这个类是最上层使用的重要的一个类，动画的绘制就是由这个类来实现。

**LottieAnimationView（绘制）**

操作集合，LottieAnimationView 继承自 AppCompatImageView，封装了一些动画的操作，具体的绘制时委托为 LottieDrawable 完成的。

数据的解析，主要参考 `LottieComposition.fromJsonSync` 方法：

```java
    static LottieComposition fromJsonSync(Resources res, JSONObject json) {
      Rect bounds = null;
      float scale = res.getDisplayMetrics().density;
      int width = json.optInt("w", -1);
      int height = json.optInt("h", -1);

      if (width != -1 && height != -1) {
        int scaledWidth = (int) (width * scale);
        int scaledHeight = (int) (height * scale);
        bounds = new Rect(0, 0, scaledWidth, scaledHeight);
      }

      long startFrame = json.optLong("ip", 0);
      long endFrame = json.optLong("op", 0);
      int frameRate = json.optInt("fr", 0);
      LottieComposition composition =
          new LottieComposition(bounds, startFrame, endFrame, frameRate, scale);
      JSONArray assetsJson = json.optJSONArray("assets");
      parseImages(assetsJson, composition);
      parsePrecomps(assetsJson, composition);
      parseLayers(json, composition);
      return composition;
    }
```

还有 `parseImages`、`parsePrecomps`、 `parseLayers` 这几个方法。


### 五. lottie vs keyframes

`lottie`由 Airbnb 出品，而`keyframes`由 facebook 出品，这两个库实现效果都差不多。据 lottie 官网说功能比 keyframes 强大一些。感兴趣的看官可以去深入研究一下。

关于 keyframes 的介绍，请参考：

[https://facebookincubator.github.io/Keyframes/](https://facebookincubator.github.io/Keyframes/)

### 六. 后续思考

##### 1. 更酷的用户引导

App的用户引导完成可以用 lottie 来实现，界面的图形可以跟随手势滑动而变化，这样的体验会更好，更新颖。

##### 2. 动效工具

完全可以开发一个预览动效的工具，提供给视觉设计师，比如，带一个二维码扫描功能，设计师设计好动效后，用这个app扫描二维码，把动画描述文件下载到本地，就可以立即看到动画在App中的效果是什么样。

##### 3. 更多动效成为可能

基于这样的动画框架，App中可以实现更多更丰富的动画效果，这将会大大提升用户体验。