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
这个例子中的各个View相互影响，却被和谐的组织在了一起。这就是使用｀CoordinatorLayout｀最简单的实例：
```xml
<?xml version="1.0" encoding="utf-8"?>

<android.support.design.widget.CoordinatorLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:background="@android:color/background_light"
    android:fitsSystemWindows="true"
    >

    <android.support.design.widget.AppBarLayout
        android:id="@+id/main.appbar"
        android:layout_width="match_parent"
        android:layout_height="300dp"
        android:theme="@style/ThemeOverlay.AppCompat.Dark.ActionBar"
        android:fitsSystemWindows="true"
        >

        <android.support.design.widget.CollapsingToolbarLayout
            android:id="@+id/main.collapsing"
            android:layout_width="match_parent"
            android:layout_height="match_parent"
            app:layout_scrollFlags="scroll|exitUntilCollapsed"
            android:fitsSystemWindows="true"
            app:contentScrim="?attr/colorPrimary"
            app:expandedTitleMarginStart="48dp"
            app:expandedTitleMarginEnd="64dp"
            >

            <ImageView
                android:id="@+id/main.backdrop"
                android:layout_width="match_parent"
                android:layout_height="match_parent"
                android:scaleType="centerCrop"
                android:fitsSystemWindows="true"
                android:src="@drawable/material_flat"
                app:layout_collapseMode="parallax"
                />

            <android.support.v7.widget.Toolbar
                android:id="@+id/main.toolbar"
                android:layout_width="match_parent"
                android:layout_height="?attr/actionBarSize"
                app:popupTheme="@style/ThemeOverlay.AppCompat.Light"
                app:layout_collapseMode="pin"
                />
        </android.support.design.widget.CollapsingToolbarLayout>
    </android.support.design.widget.AppBarLayout>

    <android.support.v4.widget.NestedScrollView
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        app:layout_behavior="@string/appbar_scrolling_view_behavior"
        >

        <TextView
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:textSize="20sp"
            android:lineSpacingExtra="8dp"
            android:text="@string/lorem"
            android:padding="@dimen/activity_horizontal_margin"
            />
    </android.support.v4.widget.NestedScrollView>

    <android.support.design.widget.FloatingActionButton
        android:layout_height="wrap_content"
        android:layout_width="wrap_content"
        android:layout_margin="@dimen/activity_horizontal_margin"
        android:src="@drawable/ic_comment_24dp"
        app:layout_anchor="@id/main.appbar"
        app:layout_anchorGravity="bottom|right|end"
        />
</android.support.design.widget.CoordinatorLayout>
```
看一下上面Layout的结构，`CoordinatorLayout`包含三个子View：
一个`AppbarLayout`，一个scrolleable View，一个指定了⚓️锚点的`FloatingActionBar`。









首发简书
首发YoungPeanut