---
layout: post
title:  "Build CryEngine From Source Code"
date:   2018-03-14 11:03:31 +0800
categories: computer-graphics
---

SVOTI(动态全局光照)是CryEngine里一个亮点。为了一窥SVOTI的真谛，让我们开始折腾起来。

## 安装CryEngine和了解SVOTI

1. 按着[Tutorials - Getting Started], 下载CryEngine Launcher. 
2. 安装CryEngine(目前版本5.4)
3. 下载GameSDK资源
4. 在CryEngine Launcher里以Editor(SandBox)打开Game SDK场景
5. 按照[Doc - Voxel-Based Global Illumination]文档，简单的了解一下SVOGI的功能
    * tool->level-settting->Global Illumination_v2->Active开启SVOTI功能
    * e_svoDebug开始调试模式
    * r_ShowRenderTarget svo_fin显示svoti中间结果
    * 在e_svoDebug模式下，新添加物件或移动，物件没有调试的矩形线框，可以勾选Global Illumination->Update Geometry

建议先看看[Beginers Guide],了解一下CryEngine's SandBox Editor一些基本使用方法。

## 编译和构建CryEngine ##

1. 按照[[Download Source Code from Github]下载CryEngine源码
2. 按照[Using C++ Project Template With Custom Engine]文档，下载和编译CryEngine.
    * 文档相对比较滞后，CryEngine目前支持CMake编译构建
    * system.cfg, engine目录需要从CryEngine Launcher的目录拷贝到源码根目录
    * 需要指定加载的工程：-project C:/FirstPersonShooter/Game.cryproject, 在我机器是黑屏，
    * [Using GameSDK with Custom Engine]中提示，+map woodland 可以指定场景的哪个Level
    
3. 修改源码，将e_svoDebug等相关变量开启，F5启动，GameSDK帧率太低
4. 按照[Beginers Guide]教程，搭建简单的场景环境，来了解SVOTI底层实现
    * SpawnPoint需要设定，不设定一般启动画面在海里。
    * SpawnPoint不在Components->Game，而在Legals Entity->Others
    * 需要配置下Flow图
5. 接下来，就是耐心的看代码

最终跑起来的画面如下：
![SVOTI](/assets/images/global_illumination/ce_svoti.jpg "SVOTI")

## 小结 ##

CryEngine只开源了引擎代码，SandBox编辑器源码未开放。如果SandBox也开源了，是可以少很多折腾的。
文档相对滞后，需要较长时间才能编译运行源码。相反，UE4则比较容易编译构建。


[Tutorials - Getting Started]:https://www.cryengine.com/tutorials/getting-started
[Doc - Voxel-Based Global Illumination]:http://docs.cryengine.com/display/CEMANUAL/Voxel-Based+Global+Illumination
[Beginers Guide]:http://docs.cryengine.com/display/CEMANUAL/Beginners+Guide
[Download Source Code from Github]:http://docs.cryengine.com/display/CEPROG/Getting+Started+With+git
[Using C++ Project Template With Custom Engine]:(http://docs.cryengine.com/pages/viewpage.action?pageId=25529768)
[Using GameSDK with Custom Engine]:http://docs.cryengine.com/display/CEPROG/Using+GameSDK+with+Custom+Engine