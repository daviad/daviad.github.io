---
layout: post
title:  "xcode 集成 unity3d"
categories: JavaScript
tags:  xcode unity
excerpt: xcode 集成 unity3c
---

* content
{:toc}

本文主要记录，xcode 集成 unity 3d：如何将unity导出的xcode的工程（取名UnityProj）添加到已有工程中（NativeProj）；  
* 方案一：源文件导入  
* 方案二：将unity导出的xcode工程的，作为当前工程的依赖工程。
* 方案三：将unity做成framework。


xcode version:8.2.1  
unity version:5.5.1f1
  
#### 方案一原理：对比UnityProj和NativeProj的不同。
主要是 
* 1.添加framework
* 2.search path
* 3.other flag
* 4.修改NativeProj 的main文件
* 5.修改Appdelegate文件
* 6.添加run script
  
  分析UnityProj可知：分析main.mm可知：程序启动初始化之后，将进入UnityAppController类。
  UnityAppController 实现了 <UIApplicationDelegate>协议。
  所以UnityProj需要由这个UnityAppController来完成NativeProj的APPdelegte中的相关功能。比如app的生命周期。launch，becomeActive等等具体看下面的代码。
  在类中创建了一个Window，以后所有的UI都是在这个window上的。这个原生的ios工程一致。


  所以，需要完成两个任务：
  * 1.创建Unity Window，和native window 交换
  * 2.管理Unity和native 的生命周期。

  
据说可以整合后只增加18mb

分析unityProj：
主要完成了一下几个任务
一、启动时。
1.初始化runtime ，及Log等   见在main.mm 
2.初始化ApplicationNoGraphic  
3.选择RenderAPI （OpengGLES，Metal）
4.创建Window，rootView，rootViewController，DisplayManager 等。
5.显示Window 并 截屏 显示
二、become active 时。
1.remove 截屏View
2.初始化Application
3.LoadApplication
4.初始化Profiler （）
5.显示在启动时创建的UI。showGameUI
6.设置Unity的player ，开启unity 循环等等。


分析代码发现：这个和原生的Window，View，的UI结构基本一样。只是使用Metal或OpenGL，Unity的player来渲染了。这个和 开发OpenGLES 类似。
在 UnityAppController 中，开启了渲染定时器：createDisplayLink； 跟踪代码发。最终主要的渲染由Unity的player 完成的。方法是extern "C" void UnityRepaint()。


所以，可以，将Unity 封装成一个View，在native中使用。当然，有些代码估计要对应修改。







参考网站：  
[http://www.cnblogs.com/fuunnyy/p/6227740.html](http://www.cnblogs.com/fuunnyy/p/6227740.html)  
[http://www.cnblogs.com/ymonke/p/5676981.html](http://www.cnblogs.com/ymonke/p/5676981.html)


Did you integrate and then compile as a dynamic framework or did you compile direct from the Unity export?
Yes,I did all。

I think That is  too complex to  read 。 My steps is easy!    Can you give me your email ，or social account：WeChat ,and so  on 。 i send you step and demo afternoon！ 

and where are you from? 



1.通过Unity 导出Xcode 工程 called UnityProject    用xcode新建一个工程（app或者是Framework）called Nativeproject
2.将Classes 和 Libraries 文件夹以 group 的方式导入自己的 NativeProject 工程中
3.将Data 文件夹以folder references 的方式导入自己的 NativeProject 工程中
4.添加framework and libraries  参考UnityProject
5。添加Header search path  参考UnityProject
6.添加 Library search path 参考UnityProject
7.在other C Flags 中添加 -DINIT_SCRIPTING_BACKEND=1 同是在 other C++ Flags中出现
8.在user-Defined  参考参考UnityProject
 添加如下 GCC_THUMB_SUPPORT NO
GCC_USE_INDIRECT_FUNCTION_CALLS NO
UNITY_RUNTIME_VERSION 5.4.0f3
UNITY_SCRIPTING_BACKEND il2cpp
9.添加 RUN SCRIPT  参考UnityProject
10.修改main.mm
11.修改  prefix
key import ： comepare  UnityProject and Nativeproject。
 
http://code4app.com/home.php?mod=space&uid=822721&do=blog&id=790
