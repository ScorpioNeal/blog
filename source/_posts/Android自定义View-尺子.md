title: Android自定义View-尺子
date: 2015-09-07 11:34:35
categories: Android
tags:
- Android
- View
---


### 代码: ###
	
	https://github.com/ScorpioNeal/RulerView
	
### 效果: ###
![img](http://7lrzgb.com1.z0.glb.clouddn.com/Screenshot_2015-09-07-11-37-00.png)

### 代码解析: ###

尺子是由`底部的矩形`， `最中间的指示器`， `几个长的刻度和几个小的刻度以及长刻度对应的值`构成。
首先可以把它们分成3个部分进行绘制， 按照上下层次，依次绘出.

```java
    @Override
    protected void onDraw(Canvas canvas) {
        super.onDraw(canvas);

        drawBackground(canvas);

        drawIndicator(canvas);

        drawLinePartition(canvas);
    }
```

画背景和指示器都很简单， 下面主要介绍下画第三部分。

1. 画长刻度
2. 画短刻度
3. 画刻度值

很容易可以发现， 长刻度只要固定下来位置， 短刻度就很容易确定位置

	短刻度_0i = 长刻度_0 + i * 短刻度间距

同理，刻度值更容易确定位置, 因为它就在长刻度的下方。

（以上位置指的水平位置）


下面就重点分析如何去画长刻度.

核心观点就是长刻度确定初始位置后

1. 根据实际偏移量来计算出相对偏移量。 相对偏移量 = 实际偏移量 - (int)(实际偏移量 / 长刻度间隔) * 长刻度间隔;
2. 根据相对偏移量绘制移动后的长刻度位置
3. 通过1中公式可以算出移动了多少个刻度，然后绘制计算后的值
4. 这样给人的感觉就是尺子的感觉了

`talk is cheap, show ur my code.`


```java
        //计算半个屏幕能有多少个partition
        int halfCount = (int) (mWidth / 2 / mPartitionWidth);
        //根据偏移量计算当前应该指向什么值
        mCurrentValue = mOriginValue - (int) (mMoveX / mPartitionWidth) * mPartitionValue;
        //相对偏移量是多少, 相对偏移量就是假设不加入数字来指示位置， 范围是0 ~ mPartitionWidth的偏移量
        mOffset = mMoveX - (int) (mMoveX / mPartitionWidth) * mPartitionWidth;

        if (null != listener) {
            listener.onValueChange(mCurrentValue, -(int) (mOffset / (mPartitionWidth / mSmallPartitionCount)));
        }

        // draw high line and  short line
        for (int i = -halfCount - 1; i <= halfCount + 1; i++) {
            int val = mCurrentValue + i * mPartitionValue;
            //只绘出范围内的图形
            if (val >= mStartValue && val <= mEndValue) {
                //画长的刻度
                float startx = mWidth / 2 + mOffset + i * mPartitionWidth;
                if (startx > 0 && startx < mWidth) {
                    canvas.drawLine(mWidth / 2 + mOffset + i * mPartitionWidth, 0 + mLineTopMargin,
                            mWidth / 2 + mOffset + i * mPartitionWidth, 0 + mLineTopMargin + mHighLineHeight, mHighLinePaint);

                    //画刻度值
                    canvas.drawText(val + "", mWidth / 2 + mOffset + i * mPartitionWidth - mIndicatorTxtPaint.measureText(val + "") / 2,
                            0 + mLineTopMargin + mHighLineHeight + mIndicatorTextTopMargin + Utils.calcTextHeight(mIndicatorTxtPaint, val + ""), mIndicatorTxtPaint);
                }

                //画短的刻度
                if (val != mEndValue) {
                    for (int j = 1; j < mSmallPartitionCount; j++) {
                        float start_x = mWidth / 2 + mOffset + i * mPartitionWidth + j * mPartitionWidth / mSmallPartitionCount;
                        if (start_x > 0 && start_x < mWidth) {
                            canvas.drawLine(mWidth / 2 + mOffset + i * mPartitionWidth + j * mPartitionWidth / mSmallPartitionCount, 0 + mLineTopMargin,
                                    mWidth / 2 + mOffset + i * mPartitionWidth + j * mPartitionWidth / mSmallPartitionCount, 0 + mLineTopMargin + mShortLineHeight, mShortLinePaint);
                        }
                    }
                }

            }

        }
```