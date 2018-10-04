---
layout: post
title:  "Global Illumination(全局光照)"
date:   2018-03-14 11:03:31 +0800
categories: computer-graphics
---

最近，花了一点时间调研了实时动态GI的技术，下面是调研过程中的一点收获。

![Global_illumination](/assets/images/global_illumination/gi_gicourse2010.jpg "gi_gicourse2010")

根据[Wikipedia]，上图红色框内的光被红色的书籍反射到龙模型和桌上水杯的焦散就是Global Illumination。接着，来看看计算机渲染的GI效果图。

![NVIDIA VXGI](/assets/images/global_illumination/ue_vxgi.jpg "NVIDIA VXGI")

上图是NVidia的VXGI的效果图。接下来，看看UE和CE两大商业引擎在动态GI上的策咯。

UE4用Lightmass来实现静态GI,LPV实现动态GI。 LPV动态GI目前还处于开发阶段。
早期的UE4用了SVOGI技术实现了效果明显的动态GI，但由于一些原因，SVOGI被舍去了。
UE4's GI的第三插件有[NVIDIA VXGI]和[Enlighten]。

LPV's GI是CryEngine研发出来实现动态GI的。目前，CryEngine用了一个SVOTI来实现动态GI。
相对于其他动态GI技术，SVOTI在性能和效果上都很有优势。

[extremeistan blog]收集很多实时GI渲染的连接，有兴趣可以去看看。

SVOGI舍去的原因可以参考UE的论坛话题[svogi]。[svogi]里有许多干货，如:

![svogi](/assets/images/global_illumination/ue_svogi.jpg "SVO")
![GI Tech Compare](/assets/images/global_illumination/gi_tech_compare.jpg "GI Tech Compare")

对于想研究动态GI技术的人而言，目前svoti技术会比较适合。因为CE源码开源了，理论相关的学术论文也很容易在网上下载到。

注：以上图片皆来源于资源连接里。

## 参考

* [Wikipedia - Global Illumination](https://en.wikipedia.org/wiki/Global_illumination)
* [Global Illumination Across Industries](http://cgg.mff.cuni.cz/~jaroslav/gicourse2010/)
* [NVIDIA VXGI](https://developer.nvidia.com/vxgi)
* [Unreal - Lightmass Global Illumination](https://docs.unrealengine.com/en-us/Engine/Rendering/LightingAndShadows/Lightmass)
* [Unreal - Light Propagation Volumes](https://docs.unrealengine.com/en-us/Engine/Rendering/LightingAndShadows/LightPropagationVolumes)
* [Unreal - Enlighten](https://www.unrealengine.com/en-US/showcase/enlighten-real-time-global-illumination-in-unreal-engine-4?sessionInvalidated=true)
* [CryEngine 3 - Light Propagation Volumes in CryEngine 3](http://www.crytek.com/cryengine/cryengine3/presentations/light-propagation-volumes-in-cryengine-3)
* [CryEngine 3 - Voxel-Based Global Illumination](http://docs.cryengine.com/display/CEMANUAL/Voxel-Based+Global+Illumination)
* [extremeistan blog- Realtime Global Illumination techniques collection](https://extremeistan.wordpress.com/2014/05/11/realtime-global-illumination-techniques-collection/)

[Wikipedia]:https://en.wikipedia.org/wiki/Global_illumination
[NVIDIA VXGI]:https://developer.nvidia.com/vxgi
[extremeistan blog]:https://extremeistan.wordpress.com/2014/05/11/realtime-global-illumination-techniques-collection/
[Enlighten]:https://www.unrealengine.com/en-US/showcase/enlighten-real-time-global-illumination-in-unreal-engine-4?sessionInvalidated=true
[svogi]:https://forums.unrealengine.com/unreal-engine/feedback-for-epic/659-svogi
