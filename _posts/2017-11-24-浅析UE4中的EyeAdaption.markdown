---
layout: post
title:  "浅析UE4中的EyeAdaption"
date:   2017-11-14 22:03:31 +0800
categories: [computer-graphics, EyeAdaptation]
description: UE4是采用基于Histogram的方法来计算
---

## EyeAdaptation In UE4

[Eye Adaptation - UE4 Documentation]中，UE4是采用基于Histogram的方法来计算画面的平均中灰值的，并通过*LowPercent*和*HighPercent*来过滤一些极黑或者极亮的
像素块。通过*Min Brightness*和*Max Brightness*来限制中灰值*L*的取值范围。*Speed Up*和*Speed Down*来控制明到暗和暗到明的过度变化。并允许通过*Auto Exposure Bias*
来曝光补偿。

#### Auto Exposure Mathematic Model

* ![Auto Exposure]({{"/assets/images/eye_adaptation/5.svg"|relative_url}} "Exposure")

* ![Exposure Calculation]({{"/assets/images/eye_adaptation/4.svg"|relative_url}} "Exposure Calculation")

* ![cache old luminance]({{"/assets/images/eye_adaptation/3.svg"|relative_url}} "Cache Old Luminance")

* ![Temporal Adaptation]({{"/assets/images/eye_adaptation/2.svg"|relative_url}} "Temporal Adaptation")

* ![Average Luminance]({{"/assets/images/eye_adaptation/6.svg"|relative_url}} "Average Luminance")

在[EyeAdaption - Overview]中，我们总结了EyeAdaptation的Mathematic Model。通过阅读UE4的Source Code，整体的EyeAdaption的过程大致一样，
只是在计算平均亮度值之一块有较大的差异。

#### Auto Exposure in UE4

* ![Auto Exposure]({{"/assets/images/eye_adaptation/5.svg"|relative_url}} "Exposure")

* ![Exposure Calculation]({{"/assets/images/eye_adaptation/7.svg"|relative_url}} "Exposure Calculation")

* ![cache old luminance]({{"/assets/images/eye_adaptation/3.svg"|relative_url}} "Cache Old Luminance")

* ![Temporal Adaptation]({{"/assets/images/eye_adaptation/11.svg"|relative_url}} "Temporal Adaptation")
* ![Temporal Adaptation]({{"/assets/images/eye_adaptation/10.svg"|relative_url}} "Temporal Adaptation")
* ![Temporal Adaptation]({{"/assets/images/eye_adaptation/9.svg"|relative_url}} "Temporal Adaptation")
* ![Temporal Adaptation]({{"/assets/images/eye_adaptation/8.svg"|relative_url}} "Temporal Adaptation")

* ![Average Luminance]({{"/assets/images/eye_adaptation/12.svg"|relative_url}} "Average Luminance")

在UE4中，计算平均中灰值的方式是基于histogram的，并且多了个曝光补偿因子来控制最终的曝光值。

## 浅析UE4中的EyeAdaption

UE4的EyeAdaption的代码基本上是按照Auto Exposure in UE4编写。因此，接下来只是单纯的把对应的mathematic equation和相关的Code放到一起，
并简单的在UE4的Code中简单注释公式和代码相关变量的关系。

#### Eye Adaptation参数

```
/// PostProcessHistogramCommon.usf

// Lmin: EyeAdaptationMin, LMax: EyeAdaptationMax
// [0] .x:ExposureLowPercent/100, .y:EyeAdaptationHighPercent/100, .z:EyeAdaptationMin, .w:EyeAdaptationMax

// temporal adaptation's *delta time*: DeltaWorldTime, temporal adaptation's *speed*: EyeAdaptionSpeedUp/Down
// [1] .x:exp2(ExposureOffset), .y:DeltaWorldTime, .zw: EyeAdaptionSpeedUp/Down

// 基于Histogram的Average Luminance的参数
// [2] .x:Histogram multiply, .y:Histogram add, .z:HistogramMinIntensity w:unused
float4 EyeAdaptationParams[3];
```

#### Eye Adaptation

![Auto Exposure]({{"/assets/images/eye_adaptation/5.svg"|relative_url}} "Exposure")

```
/// PostProcessTonemap.usf

// Exposure存储在EyeAdaptation纹理里，vEyeAdapationVS.x即Exposure
vEyeAdapationVS.x = EyeAdaptation.Load(int3(0, 0, 0)).r;
OutColor = TonemapCommonPS(UV, vEyeAdapationVS, GrainUV, FringeUV, FullViewUV, SvPosition);

// TonemapCommonPS
float ExposureScale = InExposureScaleVignette.x;
...
// LinearColor, 即Color(i,j)
LinearColor *= ExposureScale;
...

```

从EyeAdaptation纹理中得到当前的*Exposure*值，并把当前的*LinearColor*乘以该*Exposure*值来调整画面的明暗。

#### Temporal Adaptation

![Temporal Adaptation]({{"/assets/images/eye_adaptation/11.svg"|relative_url}} "Temporal Adaptation")

![Temporal Adaptation]({{"/assets/images/eye_adaptation/10.svg"|relative_url}} "Temporal Adaptation")

![Temporal Adaptation]({{"/assets/images/eye_adaptation/9.svg"|relative_url}} "Temporal Adaptation")

![Temporal Adaptation]({{"/assets/images/eye_adaptation/8.svg"|relative_url}} "Temporal Adaptation")

```
/// PostProcessEyeAdaptation.usf

float ComputeEyeAdaptation(float OldExposure, float TargetExposure, float FrameTime)
{
    float Diff = TargetExposure - OldExposure;
    
    // dark adaption & light adaption 的 *speed*参数
    const float EyeAdaptionSpeedUp   = EyeAdaptationParams[1].z;
    const float EyeAdaptionSpeedDown = EyeAdaptationParams[1].w;
    
    // Diff的正负表示从明到暗或者暗到明的过程
    float AdaptionSpeed = (Diff > 0) ? EyeAdaptionSpeedUp : EyeAdaptionSpeedDown;
    
    float Factor = 1.0f - exp2(-FrameTime * AdaptionSpeed);
    
    // Temporal Exposure *L*: OldExposure + Diff * Factor
    // clamp(L, LMin, LMax), LMin：EyeAdaptationParams[0].z， LMax: EyeAdaptationParams[0].w
    return clamp(OldExposure + Diff * Factor, EyeAdaptationParams[0].z, EyeAdaptationParams[0].w);
}
```

#### Exposure Calculation 

![Exposure Calculation]({{"/assets/images/eye_adaptation/7.svg"|relative_url}} "Exposure Calculation")

```
/// PostProcessEyeAdaptation.usf

float2 MainEyeAdaptationCommon()
{
    float2 OutColor = 0;
    // 曝光补偿
    float ExposureOffsetMultipler = EyeAdaptationParams[1].x;
    
    // 从Histogram中计算画面的中灰值
    float TargetExposure = ComputeEyeAdaptationExposure(PostprocessInput0);
    
    // Lold
    float OldExposureScale = PostprocessInput0.Load(int3(0, 1, 0)).r;
    float OldExposure = ExposureOffsetMultipler / ( OldExposureScale != 0 ? OldExposureScale : 1.0f );
    
    // delta time in temporal adaptation
    float FrameTime = EyeAdaptationParams[1].y;
    
    // eye adaptation changes over time
    float SmoothedExposure = ComputeEyeAdaptation(OldExposure, TargetExposure, FrameTime);
    
    // KeyValue = 1.0f, smoothedExposure = clamp(L, Lmin, Lmax)
    float SmoothedExposureScale = 1.0f / max(0.0001f, SmoothedExposure);
    float TargetExposureScale =   1.0f / max(0.0001f, TargetExposure);
    
    // Exposure
    OutColor.r = ExposureOffsetMultipler * SmoothedExposureScale;
    OutColor.g = ExposureOffsetMultipler * TargetExposureScale;
    
    return OutColor;
}
```

*1.0/max(0.0001f,SmoothedExposure)*避免除以一个很小的数值。

#### Average Luminance

![Average Luminance]({{"/assets/images/eye_adaptation/12.svg"|relative_url}} "Average Luminance")

```
/// PostProcessHistogramCommon.usf

float ComputeEyeAdaptationExposure(Texture2D HistogramTexture)
{
    float HistogramSum = ComputeHistogramSum(HistogramTexture);
    // low percent: EyeAdaptationParams[0].x, high percent: EyeAdaptationParams[0].y
    float UnclampedAdaptedLuminance = ComputeAverageLuminaneWithoutOutlier(HistogramTexture, HistogramSum * EyeAdaptationParams[0].x, HistogramSum * EyeAdaptationParams[0].y);
    // Lmin: EyeAdaptationParams[0].z, Lmax: EyeAdaptationParams[0].w
    float ClampedAdaptedLuminance = clamp(UnclampedAdaptedLuminance, EyeAdaptationParams[0].z, EyeAdaptationParams[0].w);
    
    return ClampedAdaptedLuminance;
}
```

可以这么理解，把画面的所有像素放到一个数组里，根据像素的Lum值大小排序，那么*HistogramSum*就是该数组的大小，[histogramSum * EyeAdaptationParams[0].x, HistogramSum * EyeAdaptationParams[0].y]表示对
画面平均亮度值有贡献的区域。即平均亮度值是该区域的亮度和除以区域所包含的像素个数。

基于Histogram的方法的Average Luminance算法涉及的内容比较多，在这里只是一带而过。

## 小结

**EyeAdaptation关键点就是如何计算画面的平均亮度值。**

[Average Luminance Calculation Using a Compute Shader](with code in blog)中，先将画面转到lum空间，通过Dx的Mimap或者Compute Shader计算出平均亮度值。
UE4也是通过Compute Shader计算出画面的Histogram，通过Histogram计算出平均亮度值。

## Reference

* [Eye Adpation(Auto-Exposure)](https://docs.unrealengine.com/latest/INT/Engine/Rendering/PostProcessEffects/AutomaticExposure/)
* [Automatic exposure](https://knarkowicz.wordpress.com/2016/01/09/automatic-exposure/)
* [Average Luminance Calculation Using a Compute Shader](https://mynameismjp.wordpress.com/2011/08/10/average-luminance-compute-shader/)
* PostProcessEyeAdaptation.usf
* PostProcessTonemap.usf
* PostProcessHistogramCommon.usf

[Eye Adaptation - UE4 Documentation]: https://docs.unrealengine.com/latest/INT/Engine/Rendering/PostProcessEffects/AutomaticExposure/
[Average Luminance Calculation Using a Compute Shader]: https://mynameismjp.wordpress.com/2011/08/10/average-luminance-compute-shader/