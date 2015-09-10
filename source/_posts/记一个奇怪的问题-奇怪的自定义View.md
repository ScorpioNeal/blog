title: 记一个奇怪的问题-奇怪的自定义View
date: 2015-09-10 22:04:22
categories: Android
tags:
- Android
- Bug
---

### 问题描述 ###
在使用自定义View的时候，在不同机器上表现有区别。 比如，画一个空心的圆

```java


        mBackStrokePaint = new Paint(Paint.ANTI_ALIAS_FLAG);
        mBackStrokePaint.setColor(mBackStrokeColor);
        mBackStrokePaint.setStyle(Paint.Style.STROKE);
        mBackStrokePaint.setStrokeWidth(mCircleStrokeWidth);
        
		Path path = new Path();
        path.addCircle(mDrawingArea.centerX(), mDrawingArea.centerY(), mRadius, Path.Direction.CCW);
        canvas.drawPath(path, mBackStrokePaint);

```

主要就是设置style为`STROKE`. 这样就是空心圆， 但是在测试中发现。 

* 在MotoX 1085 Android5.0上表现正常，是一个空心圆
* 在小米4, 小米Note, 小米Note顶配版上 只有5.0系统上表现正常， 在4.4 上有时候会是空心圆， 有时候会是实心圆， 同一个程序，同一个手机， 进入切出再进入都会显示不一样。 但是在测试demo中却没问题.

### 解决方法 ###
关闭硬件加速`android:hardwareAccelerated="false"` 

### 原因 ###
我也不知道，求指教

