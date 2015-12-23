---
layout: post
title: android-[译]掌握CoordinatorLayout
tags: android 掌握CoordinatorLayout
categories: android
---


<div class="toc"></div>


在[Google I/O 15](https://www.youtube.com/watch?v=7V-fIGMDsmE)上，谷歌发布了一个新的[ support library](http://android-developers.blogspot.com.es/2015/05/android-design-support-library.html)，里面包含了一些遵循[Material Design's spec](https://www.google.com/design/spec/material-design/introduction.html)的UI组件，比如，`AppbarLayout`, `CollapsingToolbarLayout` 和 `CoordinatorLayout`。
这些组件配合起来使用可以产生强大的效果，那么让我们通过这篇文章来学习如何使用这些组件。

###CoordinatorLayout
从名字可以看出，这个ViewGroup是用来协调它的子View的。看下图：
![CoordinatorLayout](http://androcode.es/wp-content/uploads/2015/10/simple_coordinator.gif)
这个例子中的各个View相互影响，却被和谐的组织在了一起。










首发简书
首发YoungPeanut