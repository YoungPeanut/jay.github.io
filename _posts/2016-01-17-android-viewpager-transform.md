---
layout: post
title: android-[译]两行代码搞定ViewPager的过渡动画
tags: android ViewPager PageTransformer
categories: android
---


<div class="toc"></div>

ViewPager自带了一种默认的页面滑动切换动画，但是如果产品想要更炫的滑动效果的时候怎么办呢？不要怕，我们可以使用[support library](http://developer.android.com/tools/extras/support-library.html)的[ PagerTransformer](http://developer.android.com/reference/android/support/v4/view/ViewPager.PageTransformer.html),API11 (Honeycomb) 及以上android版本都支持这个类。
用法很方便，

```
viewpager.setPageTransformer(false, new ViewPager.PageTransformer() {
    @Override
    public void transformPage(View page, float position) {
        // do transformation here
        }
});
```

`transformPage(View page, float position)`方法有两个参数，page参数代表当前view 或 fragment，position参数就是它的位置的值。滑动的时候，起始page和目标page的各自的transformPage()就会被同时触发调用。一个page的position为0代表它处于中间，为1代表它完全处于右边，为－1代表它完全处于左边。

官方文档是这样说的，position是一个page相对于屏幕中心的位置。position的值跟随用户滑动page而变化。当page填充屏幕完全可见的时候，它的position是0；page位于屏幕右边，它的position是1。两个page同时滑动到一半的时候，左边page的position是－0.5，右边page的position是0.5。（因为左右是对称的）所以，为了不考虑正负值，我们取position的绝对值：

```
final float normalizedposition = Math.abs(Math.abs(position) - 1);
```

现在左右两个page各自有了一个0到1之间的normalizedposition值(左边在递减[1,0]，右边在递增[0,1])，怎么用就靠你了。比如，首先，我们可以做一个淡入淡出效果。

```
@Override
public void transformPage(View page, float position) {
    final float normalizedposition = Math.abs(Math.abs(position) - 1);
    page.setAlpha(normalizedposition);
}
```

![fade](http://andraskindler.com/img/post/viewpager_pagertransformer_alpha.gif)

尺寸大小变化效果：

```
@Override
public void transformPage(View page, float position) {
final float normalizedposition = Math.abs(Math.abs(position) - 1);
    page.setScaleX(normalizedposition / 2 + 0.5f);
    page.setScaleY(normalizedposition / 2 + 0.5f);
}
```

![scale](http://andraskindler.com/img/post/viewpager_pagertransformer_scale.gif)

最后一个例子是使page沿Z轴方向旋转30度：

```
@Override
public void transformPage(View page, float position) {
    page.setRotationY(position * -30);
}
```

![](http://andraskindler.com/img/post/viewpager_pagertransformer_cover_flow.gif)

上面是一些简单的例子，[这里](http://stuff.mit.edu/afs/sipb/project/android/docs/training/animation/screen-slide.html#pagetransformer)官方给了两个例子。也可以把PagerTransformer和PagerTitleStrip／ PagerTabStrip结合在一起。

* 原文 [http://andraskindler.com/blog/2013/create-viewpager-transitions-a-pagertransformer-example/](http://andraskindler.com/blog/2013/create-viewpager-transitions-a-pagertransformer-example/)

* young [YoungPeanut.github.io](http://youngpeanut.github.io/)

* 博客  [http://youngpeanut.github.io/2016-01-17/android-viewpager-transform/](http://youngpeanut.github.io/2016-01-17/android-viewpager-transform/)
