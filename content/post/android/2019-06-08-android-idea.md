---
author: ymkNK
categories: Android
date: "2019-06-08T16:49:49Z"
img: 5.jpg
subtitle: Android开发
tag: Android
title: Android开发笔记（一)
---
# 前言
最近在帮朋友优化一个安卓的应用，使用Android来实现的一个华容道的小游戏。 [项目链接](https://github.com/MikarLittle/HuaRongDao)

# Android Studio的开发项目文件结构图

主要显示的是src文件夹下的结构

		 .
		├── AndroidManifest.xml 
		├── java
		│   └── com
		│       └── example
		│           └── huarongdao
		│               ├── AboutActivity.java
		│               ├── Block.java
		│               ├── Dimension.java
		│               ├── Klotski.java
		│               ├── KlotskiMapParser.java
		│               ├── L.java
		│               ├── Level1Activity.java
		│               ├── Level2Activity.java
		│               ├── Level3Activity.java
		│               ├── Level4Activity.java
		│               ├── LevelActivity.java
		│               ├── MainActivity.java
		│               └── Screen.java
		└── res
		    ├── drawable
		    ├── drawable-v24
		    ├── layout
		    ├── mipmap-anydpi-v26
		    ├── mipmap-hdpi
		    ├── mipmap-mdpi
		    ├── mipmap-xhdpi
		    ├── mipmap-xxhdpi
		    ├── mipmap-xxxhdpi
		    ├── temp-mipmap
		    └── values


# AndroidManifest.xml
这个配置文件中主要用来说明整个app都将使用哪一些Activity，同时也可以设置有没有上面的导航栏等功能。


		<?xml version="1.0" encoding="utf-8"?>
		<manifest xmlns:android="http://schemas.android.com/apk/res/android"
		    xmlns:tools="http://schemas.android.com/tools"
		    package="com.example.huarongdao">

		    <application
		        android:allowBackup="true"
		        android:icon="@mipmap/ic_launcher"
		        android:label="@string/app_name"
		        android:roundIcon="@mipmap/ic_launcher_round"
		        android:supportsRtl="true"
		        android:theme="@style/AppTheme"
		        tools:ignore="GoogleAppIndexingWarning">
		        <activity
		            android:name=".Level1Activity"
		            android:theme="@style/AppTheme.NoActionBar" />
		        <activity
		            android:name=".Level2Activity"
		            android:theme="@style/AppTheme.NoActionBar" />
		        <activity
		            android:name=".Level3Activity"
		            android:theme="@style/AppTheme.NoActionBar" />
		        <activity
		            android:name=".Level4Activity"
		            android:theme="@style/AppTheme.NoActionBar" />
		        <activity
		            android:name=".AboutActivity"
		            android:theme="@style/AppTheme.NoActionBar" />
		        <activity
		            android:name=".LevelActivity"
		            android:theme="@style/AppTheme.NoActionBar" />
		        <activity
		            android:name=".MainActivity"
		            android:theme="@style/AppTheme.NoActionBar">
		            <intent-filter>
		                <action android:name="android.intent.action.MAIN" />

		                <category android:name="android.intent.category.LAUNCHER" />
		            </intent-filter>
		        </activity>
		    </application>

		</manifest>

# Res文件夹下的Layout

		    ├── layout
		    │   ├── activity_about.xml
		    │   ├── activity_level.xml
		    │   ├── activity_level1.xml
		    │   ├── activity_level2.xml
		    │   ├── activity_level3.xml
		    │   ├── activity_level4.xml
		    │   └── activity_main.xml

在这个文件夹中的，就是app的各个页面的布局文件，在右方有可以直接拖动添加组件，但是拖动而得到的位置，最终都不是那么的准确。
因此一般都主要在代码中体现布局，这是可以最终确定布局是什么样子的。  
目前我常用的布局有：
- RelativeLayout相对布局
- ConstraintLayout约束布局

# Res的其他配置文件

#### drawable文件夹

		    ├── drawable
		    │   ├── aboutbtn.xml
		    │   ├── angry.jpg
		    │   ├── back_icon.jpg
		    │   ├── backimage.jpg
		    │   ├── btn.xml
		    │   ├── cry.jpg
		    │   ├── ic_launcher_background.xml
		    │   ├── levelback.jpg
		    │   └── surprise_icon.jpg

这个文件夹主要用于存放图片的资源文件，可以使用xml来配置图片被点击的时候的动画（press=false或者press=true的时候分别显示哪一张图）

		<?xml version="1.0" encoding="utf-8"?>
		<selector xmlns:android="http://schemas.android.com/apk/res/android">
		    <item android:drawable="@drawable/surprise_icon"
		        android:state_pressed="true"/>
		    <item android:drawable="@drawable/back_icon"
		        android:state_pressed="false"/>
		</selector>

在上面的代码中，android:state_pressed="true"的时候（也就是受到按压的时候），就显示surprise_icon，如果为"false"就显示back_icon

#### values文件夹

		    └── values
		        ├── attrs.xml
		        ├── colors.xml
		        ├── dimens.xml
		        ├── strings.xml
		        └── styles.xml


在这些xml文件当中，可以配置Android应用中的各种属性。

		<resources>
		    <dimen name="fab_margin">16dp</dimen>
		    <dimen name="level_text_size">24sp</dimen>
		    <dimen name="level_text_size_big">30sp</dimen>
		    <dimen name="level_text_width">192dp</dimen>

		</resources>

**但是有一点需要注意的是**,在调用的时候不需要加s

       android:textSize="@dimen/level_text_size"

# Java文件夹
在这个文件夹下主要就是用来实现程序主要逻辑的部分。一般开发需要以下几个步骤
- 需要一个类Activity继承AppCompatActivity
- 重写类中的onCreate方法，将类与对应的Layout绑定（使用R.layout.\*\*__activity）
- 使用findViewById方法，可以将layout中的各个组件，通过id找到，然后和java对象进行绑定，然后进行相应的操作

#### 例子：监听TextView的点击事件

	   	final TextView aboutGame = findViewById(R.id.about);
        aboutGame.setOnTouchListener(new View.OnTouchListener() {
            @Override
            public boolean onTouch(View v, MotionEvent event) {
                switch (event.getAction()) {
                    case MotionEvent.ACTION_DOWN:
                        aboutBtn.setBackgroundResource(R.drawable.angry);
                        aboutGame.setTextColor(getResources().getColor(R.color.colorGrey));
                        break;
                    case MotionEvent.ACTION_UP:
                        aboutBtn.setBackgroundResource(R.drawable.cry);
                        aboutGame.setTextColor(getResources().getColor(R.color.colorb));
                        break;
                }
                return false;
            }
        });
