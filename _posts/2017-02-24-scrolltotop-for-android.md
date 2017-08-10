---
layout:     post
title:      "ScrollToTop for Android"
subtitle:   " \"If you fail, don't forget to learn your lesson.\" "
date:       2017-02-23 12:00:00
author:     "leehong"
header-img: "img/post-bg-android.jpg"
catalog: true
tags:
    - ScrollToTop
    - Android
---


ScrollToTop项目实现一个通用的控制页面滚回到顶部的功能，ios系统中有这个自带功能，android中实现的很少。基于这个想法，开发了这个项目。使用者可以方便的实现全局的滚回顶部控制。

### 1. 使用方式

支持了滚动到顶部的控件有：

* ScrollToTopScrollView
* ScrollToTopListView
* ScrollToTopGridView
* ScrollToTopWebView
* ScrollToTopExpandableListView。

**xml中使用**

```xml
<com.aliwx.android.scroll.ui.ScrollToTopListView
        android:id="@+id/listview"
        android:layout_width="match_parent"
        android:layout_height="match_parent"/>
```

**Java中使用**

ListView listView = new ScrollToTopListView(context);

### 2. 使用场景

* 页面的主体是ScrollView，ListView，GridView，WebView的需要使用这些控件，支持滚动到顶部。
* 页面的其中一小部分是上述控件的不需要支持。
* 对话框中不需要滚动到顶部，不使用这些控件。

### 3. 实现方式

SDK提供了两种方式滚动到顶部

##### 1. 针对具体的可滚动View

`ScrollToTopScrollView`，`ScrollToTopListView`，`ScrollToTopGridView`，`ScrollToTopWebView`，`ScrollToTopExpandableListView` 都实现了 `IScrollToTopInterface` 接口，这个接口的定义如下：

```java
public interface IScrollToTopInterface {
    void scrollToTop();
    boolean isScrollToTopEnabled();
    void setScrollToTopEnabled(boolean var1);
}
```

如果想单独使用GridView/ListView/ScrollView等滚动到顶部，可以直接调用对应view的 scrollToTop() 方法。

##### 2. 针对整体Activity

sdk中提供了`ScrollToTopHelper`类，提供了两个滚动的方法：

```java
public static View scrollToTop(final Activity activity)
 public static View scrollToTop(final ViewGroup viewGroup)
```

传递activity或者activity的根view，内部会找到实现了`IScrollToTopInterface`接口的第一个在屏幕上可见且可滚动view，并进行滚动。可以在activity的共同基类中点击titlebar调用此方法，就能实现全局的滚动到顶部。

##### 3. 类图

![](/img/2017/2017-02-24-scroll-to-top-class.png)


源码地址：

[https://github.com/leehong2005/ScrollToTop](https://github.com/leehong2005/ScrollToTop)

请关注：

* [SlideBack](https://github.com/leehong2005/SlideBack)
* [我的GitHub](https://github.com/leehong2005)
* [我的GitHub博客](https://leehong2005.github.io)

（完）



