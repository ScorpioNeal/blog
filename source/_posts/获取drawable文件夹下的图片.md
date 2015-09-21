title: 获取drawable文件夹下的图片
date: 2015-09-15 21:54:54
categories: Android
tags:
- Tips
- Android
---
在看别人的代码的时候偶然发现一段挺新奇的代码片段

#### 获取drawable文件夹下的图片资源 ####

1. 假设在drawable文件夹下原本有一下几张图片
	[main_back0.png, main_back1.png, main_back2.png, ..., main_back5.png]
	
2. 获取方法

```java

	Random random = new Random();
	try{
    	Field field=R.drawable.class.getField("main_back"+i);
    	int j= field.getInt(new R.drawable());
    	mIvBack.setBackgroundResource(j);
	}catch(Exception e){
    	mIvBack.setBackgroundResource(R.drawable.main_back1);
	}
	
```


3. 虽然没什么卵用，但是感觉这种用法挺新奇的 :) 