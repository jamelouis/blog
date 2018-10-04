---
layout: post
title:  "资源泄露问题"
date:   2018-01-10 22:03:31 +0800
categories: python
---

## 概述 ##

近期，遇到了一些资源泄露的问题。资源泄露可以粗略的分为：**内存泄露**和**显存泄露**。
内存泄露，一般是堆上的数据创建后，在程序退出时未释放导致的。显存泄露，一般是显存资源在程序
退出时未释放导致。本文假设已经存在资源泄露的情况下，如果定位泄露所在，然后分析为何泄露，给出
泄露的处理方案。

## 内存泄露 ##

内存泄露一般可以从以下几个方面来考虑：
* msdn 检测泄露的api
* 自定内存分配策略的调试支持
* 第三方库或者软件

### msdn 检测泄露的api ###

基本上是重载内存分配的操作，并把调试信息内嵌到新的重载操作。参考[chasing static memory leaks]。

![memory leak](/assets/images/memory_leak/mem-leak-output.jpg "memory leak")

### 自定内存分配策略的调试支持 ###

如果本来工程就架构了内存分配模块，那么需要开启调试功能，利用调试功能定位到资源泄露的位置。

### 第三方库或者软件 ###

**vld库**。一般需要将vld.h文件包含到项目里，重新编译后，运行程序即可。

![memory leak](/assets/images/memory_leak/vld_output.jpg "memory leak")

**Deleaker**。商业收费软件。

## 显存泄露 ##

显存泄露，对于dx11而言，一般有两种处理方式：
1. d3d debug api
2. visual studio 的start graphic debugging

### d3d debug api ###

基本思路是，给显存资源添加额外的表示信息来定位泄露的资源。
具体参考[Object Naming],[Direct3D 11 Debug API Tricks].

### start graphic debugging ###

Visual Studio支持当遇到d3d调试信息中断的机制。需要配置一下dx的属性，然后start graphic debuging来运行程序，
当遇到第一个中断类型的信息时，程序会中断下来。


![dx panel](/assets/images/memory_leak/dx-panel.jpg "dx panel")
![dx panel break](/assets/images/memory_leak/dx-panel-break.jpg "dx panel break")

## 小结 ##

一切都是引用计数的问题。

### References ###

* [chasing static memory leaks](http://blogs.microsoft.co.il/dinazil/2014/11/14/chasing-static-memory-leaks/)
* [MemTrack: Tracking Memory Allocations in C++](http://www.almostinfinite.com/memtrack.html)
* [Direct3D 11 Debug API Tricks](http://seanmiddleditch.com/direct3d-11-debug-api-tricks/)
* [Object Naming](https://blogs.msdn.microsoft.com/chuckw/2010/04/15/object-naming/)

[chasing static memory leaks]:http://blogs.microsoft.co.il/dinazil/2014/11/14/chasing-static-memory-leaks/
[Direct3D 11 Debug API Tricks]:http://seanmiddleditch.com/direct3d-11-debug-api-tricks/
[Object Naming]:https://blogs.msdn.microsoft.com/chuckw/2010/04/15/object-naming/