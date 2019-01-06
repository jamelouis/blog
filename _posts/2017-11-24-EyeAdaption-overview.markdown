---
layout: post
title:  "EyeAdaption - Overview"
date:   2017-11-24 22:03:31 +0800
categories: computer-graphics
description: 在眼睛生理学中，适应是眼睛适应各种不同层次明暗的能力
---

## EyeAdaption

回想去电影院看电影。进入影厅，常常摸黑走到自己的座位上，但过了一段时间，却又能很容易看到周围的环境，譬如朋友买的饮料。电影结束灯亮起来时，又瞬间看清了影厅的环境。

从明亮的环境到昏暗的环境，人眼慢慢适应昏暗的环境，逐渐看清昏暗里的事物，和从昏暗的环境到明暗的环境，瞬间看清明暗里的事物。这就是人眼适应。

>In ocular physiology, adaptation  is the ability of the eye to ajust to various levels of darkness and light - [Adaptation(eye) - wikipedia]

>在眼睛生理学中，适应是眼睛适应各种不同层次明暗的能力

EyeAdaption可以分为[Dark Adaptation]和[Light Adaptation], 分别是指从明到暗与从暗到明的适应过程。从生物学层面分析，人眼感受光线的范围很大，但在任意时刻只能感受相对小的范围。
感官组织cones和rods对光线的不同适应能力，导致人眼适应昏暗的时间比适应明亮的时间要长很多([Adaptation(eye) - wikipedia],[Dark Adaptation])。

![Visual Response to Darkness]({{ "/assets/images/eye_adaptation/dark_adaptation.png" | relative_url}} "Visual Response to Darkness - wikipedia")


## EyeAdaption In game

### Exposure Calculation

>In Krzysztof Narkowicz [Automatic exposure - knarkowicz] blog:
>
>Standard approach to automatic exposure is to compute scene's geometric mean of luminance(log2 average) and map it to some "key value"：
>
>![Exposure Calculation]({{ "/assets/images/eye_adaptation/4.svg" | relative_url}} "Exposure Calculation")
>
>Then we multiply all pixels by exposure, add tone mapping, color grading and gama.

>
>自动曝光的Exposure计算
>
>自动曝光的标准方法是计算画面的对数明度的几何平均值*L*，通过*[Lmin,Lmax]*限制L的范围，通过*KeyValue*映射为*Exposure*,然后画面以*Exposure*系数曝光。

### Temporal Adaptation

> In Krzysztof Narkowicz [Automatic exposure - knarkowicz] blog:
>
> ![Temporal Adaptation]({{"/assets/images/eye_adaptation/1.svg"|relative_url}} "Temporal Adaptation")

> 时间适应
>
> 在由明到暗或者暗到明的过程中，*exposure*的数值是在一段时间内是动态变化的。为了模拟[Dark Adaptation]和[Light adaptation]，需要通过时间差来过度*exposure*数值变化。

### Auto Exposure Mathematic Model

* ![Auto Exposure]({{"/assets/images/eye_adaptation/5.svg"|relative_url}} "Exposure")

* ![Exposure Calculation]({{"/assets/images/eye_adaptation/4.svg"|relative_url}} "Exposure Calculation")

* ![cache old luminance]({{"/assets/images/eye_adaptation/3.svg"|relative_url}} "Cache Old Luminance")

* ![Temporal Adaptation]({{"/assets/images/eye_adaptation/2.svg"|relative_url}} "Temporal Adaptation")

* ![Average Luminance]({{"/assets/images/eye_adaptation/6.svg"|relative_url}} "Average Luminance")

其中，*KeyValue*的默认值常设置为0.18([Middle gray - wikipedia]); 
*speed*的取值在从明到暗或暗到明的过程中取值不一样，一般从明到暗的数值会比从暗到明的数值小。

## EyeAdaptation in UE4

![Eye Adaptation in UE4]({{"/assets/images/eye_adaptation/eye_adaptation_in_ue4.gif"|relative_url}} "Eye Adaptation in UE4")

Eye Adaptation in UE4中，当相机进入椅子的阴影时，画面的明度缓缓的发生变化。当相机从阴影中出来，画面的明度较快的恢复到之前的明暗度。

## Reference

* [Eye adaptation](https://en.wikipedia.org/wiki/Adaptation_(eye))
* [Dark Adaptation](http://ucalgary.ca/pip369/mod3/brightness/darkadaptation)
* [Light Adaptation](http://ucalgary.ca/pip369/mod3/brightness/lightadaptation)
* [Automatic exposure](https://knarkowicz.wordpress.com/2016/01/09/automatic-exposure/)

[Adaptation(eye) - wikipedia]: https://en.wikipedia.org/wiki/Adaptation_(eye)
[Dark Adaptation]: http://ucalgary.ca/pip369/mod3/brightness/darkadaptation
[Light Adaptation]: http://ucalgary.ca/pip369/mod3/brightness/lightadaptation
[Automatic exposure - knarkowicz]: https://knarkowicz.wordpress.com/2016/01/09/automatic-exposure/
[Middle gray - wikipedia]: https://en.wikipedia.org/wiki/Middle_gray