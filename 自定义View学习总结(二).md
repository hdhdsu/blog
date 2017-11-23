<!--
author: 冷火-王胜 
date: 2017-2-28 
title: 自定义View学习总结(二)
tags: Android View
category: Android
status: publish 
-->
在我们的自定义学习总结的第一篇，介绍了自定义控件的基本流程，几个核心方法和核心api，这篇主要介绍给自定义控件增加属性，就像系统控件TextView有自己的属性textSize,text一样，我们自己自定义的控件也可以有自己的属性，这样就可以跟系统控件一样，在我们的xml里面能用自己定义的属性。
1.在values目录下新建attrs.xml文件

    <resources>
        <declare-styleable name="SettingItemView">
         <!-- 显示不同的title内容,这里的属性名可随意定义,但最好不要定义和现有属性一致的名称 -->
        <attr name="item_title" format="string" />
    </declare-styleable>
    </resources>
我们只定义了一个属性name，格式是string，format是值该属性的取值类型:
一共有：string,color,demension,integer,enum,reference,float,boolean,fraction,flag;

2.在activity_main.xml文件中定义名称空间和声明属性

    <LinearLayout
    android:id="@+id/activity_main"
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    ...
    >

    <com.xinguangnet.myphoto.view.SettingItemView
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        app:item_title="我的设置1">
    </com.xinguangnet.myphoto.view.SettingItemView>
请注意这句代码  xmlns:app="http://schemas.android.com/apk/res-auto"，这个类似于系统控件里的android，如果改成xmlns:ws="http://schemas.android.com/apk/res-auto"，那么下面的 app:item_title="我的设置1"就要对应改成 ws:item_title="我的设置1"
3.在构造函数初始化自定义控件的时候获取自定义属性

     // 读取自定的属性
       TypedArray ta = context.obtainStyledAttributes(set,
         R.styleable.SettingItemView);
       //获取自定义属性的值
       String title = ta.getString(R.styleable.SettingItemView_item_title);
       //获取完成后一定要做回收处理
       ta.recycle();