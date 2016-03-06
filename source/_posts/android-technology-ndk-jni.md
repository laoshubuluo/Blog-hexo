title: Android-Technology-NDK-实现JNI
description: Android开发技术：通过NDK-JNI调用本地方法
date: 2016/03/06 18:50:30 
categories: Android
tags: [Android,Technology,NDK,JNI,C,C++,本地方法,AndroidStudio]
---
甚是惭愧，搞了两年Android，居然没研究过JNI，对NDK也是一知半解。近期因为项目需要，抽时间研究了一下如何在AndroidStudio平台上，开发能够执行本地C、C++方法的Android程序，也就是所说的SDK-JNI。
<!--more-->
是的发生的发生是的发送到

如果所有activity需要统一处理，设定一个含有转场动画的theme，然后在AndroidManifes.xml描述文件中将所有activity进行theme设定，虽然能实现，但未免有重复的嫌疑。

看到AndroidManifes.xml描述文件中，application节点是包含所有activity节点的，我们知道application是用来保存全局变量的，那如果我**将theme设定在application节点**上，是不是**所有activity也能统一识别**呢？（好吧，我是蒙的）经过测试，答案是肯定的，这样就避免了重复设定所有activity的theme了。

问题虽然简单，但仍然记录下，防止自己犯同样的错误。

## 创建动画文件 ##
在anim路径下，分别创建如下动画文件：

push_left_in.xml

	<?xml version="1.0" encoding="utf-8"?>
	<set xmlns:android="http://schemas.android.com/apk/res/android">
	 <translate
	     android:duration="300"
	     android:fromXDelta="100%"
	     android:toXDelta="0" />
	</set>

push_left_out.xml

	<?xml version="1.0" encoding="utf-8"?>
	<set xmlns:android="http://schemas.android.com/apk/res/android" >
	 <translate
	     android:duration="300"
	     android:fromXDelta="0"
	     android:toXDelta="-100%" />
	</set>

push_right_in.xml

	<?xml version="1.0" encoding="utf-8"?>
	<set xmlns:android="http://schemas.android.com/apk/res/android" >
	 <translate
	     android:duration="300"
	     android:fromXDelta="-100%"
	     android:toXDelta="0" />
	</set>

push_right_out.xml

	<?xml version="1.0" encoding="utf-8"?>
	<set xmlns:android="http://schemas.android.com/apk/res/android">
	 <translate
	     android:duration="300"
	     android:fromXDelta="0"
	     android:toXDelta="100%" />
	</set>

## 创建含有动画的theme ##
在styles.xml文件中，添加如下代码

	<!-- activity 移入移出的动画效果-->
	<style name="ActivityTheme">
	    <item name="android:windowAnimationStyle">@style/ActivityInOutAnimation</item>
	</style>
	
	<style name="ActivityInOutAnimation" parent="@android:style/Animation.Activity">
	    <item name="android:activityOpenEnterAnimation">@anim/push_left_in</item>
	    <item name="android:activityOpenExitAnimation">@anim/push_left_out</item>
	    <item name="android:activityCloseEnterAnimation">@anim/push_right_in</item>
	    <item name="android:activityCloseExitAnimation">@anim/push_right_out</item>
	</style>

## 设定theme ##
在AndroidManifest.xml文件application节点下，添加如下代码

	<application
	      android:name=".application.AmenApplication"
	      android:allowBackup="true"
	      android:hardwareAccelerated="true"
	      android:icon="@mipmap/ic_launcher"
	      android:label="@string/app_name"
	      android:persistent="true"
	      android:theme="@style/ActivityTheme">

如此，统一的theme主题，就通过application应用到所有activity中了。