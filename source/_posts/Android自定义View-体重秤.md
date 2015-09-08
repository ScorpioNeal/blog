title: Android自定义View-体重秤
date: 2015-09-08 21:21:05
categories: Android
tags:
- Android
- View
---

### 效果 ###
![img](http://7lrzgb.com1.z0.glb.clouddn.com/72CEC0AA-1462-444F-AA5E-E4E0AD09E656.png)

### 功能 ###

* 自定义表盘背景色
* 自定义进度颜色
* 自定义刻度颜色
* 自定义文本颜色
* 设置刻度字体大小
* 设置文本字体大小
* 设置起始、结束角度
* 设置起始、结束值

### 用法 ###

1. 在布局中声明

```xml
	xmlns:scale="http://schemas.android.com/apk/res-auto"
```

2. 使用view, 未定义的属性使用默认的属性

```xml
    <com.scorpioneal.scaleview.ScaleView
        android:id="@+id/scaleview"
        scale:scaleBackground="@android:color/holo_red_light"
        scale:scaleSecondaryBackground="@android:color/holo_orange_light"
        android:layout_width="match_parent"
        android:layout_height="match_parent" />
```

3. 在java文件中

```java
		
		//见名字应该可以理解
        mScaleView = (ScaleView)findViewById(R.id.scaleview);
        mScaleView.setMinMaxValue(100, 400);
        mScaleView.setStartEndAngle(180, 360);
        mScaleView.setScaleBackgroundColor(color);
        mScaleView.setScaleSecondaryBackgroundColor(color);
        mScaleView.setScaleNumberColor(color);
        mScaleView.setScaleTextColor(color);
        
        //设置显示的值
        mScaleView.setShownValue();
        //设置显示的文本，比如测量中...或者数值
        mScaleView.setShowText();
        
```

### 代码 ###

[ScaleView in Github](https://github.com/ScorpioNeal/ScaleView)

### 解析 ###

`背景大圈`, `进度`, `显示的文本`都容易画出来，这里就不赘述。

稍微有点难度的就是刻度以及刻度对应的数值和指针

先把代码贴出来

#### Code ####

```java

	        //圆心
        float originX = mRadius + VIEW_PADDING;
        float originY = mRadius + VIEW_PADDING;

        //画刻度
        for(int i = 0; i < count; i++) {
            //总共7个刻度
            float angle = mStartAngle + i * (mEndAngle - mStartAngle) / (count - 1);

            if(angle <= mSweepAngle + mStartAngle){
                mScalePaint.setColor(mScaleSecondaryBackgroundColor);
            }else {
                mScalePaint.setColor(mScaleBackgroundColor);
            }

            float angleValue = (float)(angle / 180 * Math.PI);
            float startX = (float)(originX + (mRadius - mScaleLength) * Math.cos(angleValue));
            float startY = (float)(originY + (mRadius - mScaleLength) * Math.sin(angleValue));
            float endX   = (float)(originX + mRadius * Math.cos(angleValue));
            float endY   = (float)(originY + mRadius * Math.sin(angleValue));
            canvas.drawLine(startX, startY, endX, endY, mScalePaint);

            //画刻度的数字
            float textWidth = mScaleNumPaint.measureText((int)angle+"");
            float textHeight = calcTextHeight(mScaleNumPaint, (int)angle+"");

            float x = (float)(startX - mScalePadding * Math.cos(angleValue)) - textWidth / 2;
            float y = (float)(startY - mScalePadding * Math.sin(angleValue)) + textHeight / 2; //坐标系不同，所以要minus

            float value = mMinValue + (angle - mStartAngle) * (mMaxValue - mMinValue) / (mEndAngle - mStartAngle);
            canvas.drawText((int)value + "", x, y, mScaleNumPaint);
        }

        //画中下方的文字
        float txtWidth = mDescripPaint.measureText(mShowText);
        //大约在3/4的下方
        canvas.drawText(mShowText, VIEW_PADDING + mRadius - txtWidth / 2, VIEW_PADDING + 7 * mRadius / 4, mDescripPaint);

        //画指针 TODO use bitmap to replace line
        float angleValue = (float)((mStartAngle + mSweepAngle) / 180 * Math.PI);
        canvas.drawLine(originX, originY, (float)(originX + (mRadius - 120) * Math.cos(angleValue)), (float)(originY + (mRadius - 120) * Math.sin(angleValue)), mScaleProgressPaint);


```

 * 首先根据旋转的角度以及刻度长度和大圈的半径，通过三角函数计算可以得出刻度的x, y坐标，见代码中的注释
 * 获取到刻度的坐标之后， 刻度对应的数值就在刻度与圆心相连接的直线上，要注意数值本身的宽高需要计算进去
 * 最后画指针，很简单
 
#### 动画中的效果 ####

动画中有点的反弹效果模拟实际的体重秤称重时候的效果， 这个可以自定义一个加速器Interpolator

```java

    class MyIntepolator extends BounceInterpolator {
        private float bounce(float t){
            return t * t * 8.0f;
        }
        @Override
        public float getInterpolation(float t) {
//            t *= 1.1226f;
            if (t < 0.3535f) return bounce(t);
            else if (t < 0.7408f) return bounce(t - 0.54719f) + 0.7f;
            else if (t < 0.9644f) return bounce(t - 0.8526f) + 0.9f;
            else return bounce(t - 1.0435f) + 0.95f;
        }
    }
    

```

可以从网上找下[不同动画对应的不同函数是什么样的](http://my.oschina.net/banxi/blog/135633#OSC_h1_2)， 至于效果需要自己微调下。


### TODO ###

 * 现在只支持均分的，不均分暂时不支持
 * TODO 异常数据还没做处理
 * TODO 支持手势操作
 * TODO 适配
 * TODO 表针可替换