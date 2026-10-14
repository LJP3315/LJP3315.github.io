---
title: "安卓Activity生命周期"
date: 2026-10-14
categories: school
tags: Android
---

记录实验2中的知识点生命周期函数的内容

### 理解基本的生命周期方法

onCreate()：整个生命周期只会被调用一次，在界面初始化时调用

调用该方法，会进行初始化布局，绑定UI控件，初始化变量，设置监听器



onStart()：Activity即将对用户可见是调用

在屏幕可见，但尚未获得焦点的时候调用，通常发生在 onCreate 或 onRestart 之后



onResume()：界面获得焦点时调用

界面位于屏幕最前端，用户可以进行交互



onPause()：界面失去焦点时调用，界面部分可见，但无法交互

快速保存数据



onStop()：界面完全不可见时调用

释放资源，断开数据库的链接



onRestart()：界面由不可见重新变为可见前调用



onDestroy()：Activity被销毁前调用，整个生命周期只会调用一次。

彻底清理资源，放置内存泄漏

### 通过示例理解生命周期

正常启动流程：onCreate() → onStart() → onResume()

先创建，屏幕可见，获得焦点



用户切到后台：onPause() → onStop()

失去焦点，界面不可见



用户返回应用：onRestart() → onStart() → onResume()

不可见变为可见，屏幕可见，获得焦点



用户关闭当前界面：onPause() → onStop() → onDestroy()

失去焦点，界面不可见，销毁整个 Activity

### 总结

可以简单的将这些生命周期函数贴上标签，这样可以方便记忆

创建应用(onCreate)，屏幕可见(onStart)，获得焦点(onResume)，失去焦点(onPause)，屏幕不可见(onStop)

由不可见变为可见(onRestart)，销毁应用(onDestroy)

