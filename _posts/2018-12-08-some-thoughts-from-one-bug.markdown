---
layout: post
title:  一个bug引发的思考
date:   2018-12-08 11:03:31 +0800
categories: misc
---

## 概述
因伤休假2个月后回来上班，被指派去修复一个项目开发中的bug：

> 工程项目在Android Studio无法调试C/C++本地代码  

在修复bug的过程中，对工程开发有了一些感悟，比如，
* bug从哪里来，到哪里去，在bug的“一生”中留下了什么
* bug有繁衍性质吗，后序是否有相关bug的出现
* bug处理时机与处理方式和处理代价的关系
* 如何维持跨平台下开发

## Bug的“人生哲学”：我是谁
项目是个跨平台的，目前主要是移动端的，所以有Android，iOS，当然也有PC/Mac的。团队成员一般在PC的Visual Studio开发环境上做日常的开发，平常很少在Android Studio上开发测试，除非是一些涉及到Android相关的功能。Bug是在开发Android相关功能时发现的，但不影响开发，或者说不影响具体的功能开发，所以只是留了个缺陷单。

Bug据说已存活个把月了。Bug就在我们不知觉的状态，登堂入室了。但因优先级不是很高，且我之前用cmake重构过nk版的Android工程，所以bug就内定了我这个伤号。

那时对bug来了个深切的问候：

> 你是谁？你来自哪里？  

## 给Bug来个手术
古代医生看病，讲究望闻问切。修复bug，亦是如此。

**复现**。能复现的bug，就算修复了90%。剩下的10%就只是时间问题了。更新代码，开启Android Studio工程，在C_C++本地代码的常用接口处设置断点和接口调用的Java代码设置断点，确认的 bug模式为auto(可调试Java_C_C++），编译运行工程。Java断点触发，C_C++未触发，成功复现Bug。

**排除IDE问题**。在Android Studio开发环境，创建一个带有C/C++代码的工程，按照上述的方式设置断点，断点皆触发。排除是由于IDE开发环境原因。

做到了bug复现和排除了是开发环境的问题，接下来就是上网搜索相关的资料：
* [Debug your app  |  Android Developers](https://developer.android.google.cn/studio/debug/)
	* 了解基本的调试知识
* 确认CMake，LLDB，NDK是否安装
    * 非IDE问题，可确认安装
* 配置AndroidManifest属性：android:debuggable=true
    * 无效，此属性会根据构建的类型自动设置
    * 可以设置构建类型分别为debug/release，然后build-analyze apk
    * 在Analyze Apk窗口，选择AndroidManifest文件，查看对应属性
* 在build.gradle中设置：jniDebuggable = true
    * 新版本已废弃该属性
* 导入同事的setting.jar
	* 都不能调试，无法验证

做了一番尝试后，发觉竟然是个“顽疾”。在尝试的过程中，因为项目的庞大性制，每次尝试一种可能的解决方案时，在验证处理上耗时很长，所以先放下了去寻找解决方案，从另外一个维度去做减法模式。

**减法模式**。鉴于之前的重构经验，轻车熟路地把C/C++相关的代码移除，只留下一些空的接口实现，只剩一个空架子。

做完减法模式后，又重复复现的操作，可复现，又重新回到了新的起点。Ctrl-C/Ctrl-V模式失败后，打算换个角度来处理，因为之前简单的工程可以调试，所以考虑从两者的异同对比来挖掘出可能的解决方案。

 **AndroidManifest属性**。删除原来一些权限属性，尽量使两者的属性保持一致。编译运行验证，仍不可调试。

**build.gradle**。删除一些原来跨平台的资源拷贝和资源组织等非明显无关的配置属性。编译运行验证，仍不可调试。

基于上述的处理后，整个工程项目的文件基本不多了。因此，用比较工具对比两个工程文件夹，细心地进行文件的一一对比。

在app.iml文件中，有一个明显的异常，如下：
```xml
<facet type="native-android-gradle" name="Native-Android-Gradle">       
<configuration>         
<option name="SELECTED_BUILD_VARIANT" value="release" />       
</configuration>     
</facet>
```
 
手动修改后，通过另一番折腾（见下文）后，终于可以调试了。通过版本管理工具，恢复了“手术”处理现场，回到最初的状态，手动更新* SELECTED_BUILD_VARIANT*属性，编译运行验证，可调试。于此，bug算走到它的尽头了。

## bug留下了什么？
走到这里，已经算可以交差了，但细细一想，bug从哪里来，仍然还是一无所知。处理过程中，团队的其他人员有的开始感受到Android Studio无法调试带来的麻烦。在接受这个bug时，和同事或多或少聊了，对bug的由来都不知所措。而我在尝试解决这个bug时，也耗费了不少脑细胞，所以对自己提了几个问题：
* app.iml的属性是怎么被修改的
* 如何避免同样，或者类似的问题
* 假如更早的处理这个问题，会怎么样的情形？
	
### app.iml的属性是怎么被修改的
点击菜单 build -> select build variant ，会有一个面板出现在左边的工作区，有个下拉菜单可以选择 debug/release 选项。用文本编辑器打开工程项目的 app.iml 文件，找到 **native-android-gradle** 项。切换下拉菜单项，对应的app.iml的选项也跟着变化。然后手动改变属性，返回Android Studio再次切换属性选项，尝试了几次，遇到IDE报错，遇到了切换属性对应的 app.iml 配置没有变化。
因此，猜测可能是有同事本地切换了 release ，更新代码过程中，有冲突或者什么原因导致了该属性被设置成了 release 。至此，大体了解了bug来自哪里了。

### 如何避免同样，或者类似的问题
目前，bug的问题可以归结为：
> 由于 app.iml 文件的属性被设置成 release ，因此不可调试。  
从这个角度看，只要修改属性即可解决问题，但并不能保证以后不发生。

**app.iml 文件**。一番搜索后，该文件属于中间文件，可以不纳入版本控制里。这也可能是之前导入其他同事setting.jar解决问题的缘由。
> 由于把中间配置文件加入了版本控制里，因此所有开发者的开发环境都不可调试。  
从这个角度看，只要从版本控制工具里移除 .iml 文件即可解决问题，可以保证不影响其他同事的开发环境。

### 假如更早的处理这个问题，会怎么样的情形？
对于一些比较棘手、没有头绪的、让人不知所措的bug，假如之前没有的和项目使用了版本管理工具，一般的思路会是采用二分法回滚代码测试验证。

因为之前的版本是可调试的，如果是更早的处理这个bug，通过二分回滚法，会是很简单的一个bug。因为bug的产生就是配置属性被错误的修改了。而且，是越早越容易处理。

Bug存在时间已有个把月了，回溯的工作量很多，且难以保证回溯后的工程可以编译成功。因此，就没选择二分回滚大法。

## 跨平台下处理bug的一些思考
由上述可知，及时反馈及时处理可以减少很多不必要的工作和精力。但同时维护多个平台时，特别是项目开发周期很紧张时，是一件很难的事情。因此，开发者一般都是在某一特定的平台下开发跨平台的功能，只有涉及到具体平台相关的功能，才会切换到对应的开发平台下继续开发。

因为这样的开发模式，能加快功能的开发进度，但同时也引入不少的弊端，譬如文中要处理的bug，其他平台编译不过，其他平台不可运行等等。特别针对某个特定平台版本验收时，可能会暂时不去开发其他平台的功能需求，这种一般导致其他平台项目滞后，如我之前重构的nk工程，那时任务是让nk工程可运行。

可能会觉得这样的开发模式是不对的。开发某种功能，需要在多个平台下统一验证下，才可以提交对应的代码修改。或许是对的，但需要团队的统一认可和无条件的执行。目前，实际上很难做到。

**见招拆招**。引入持续集成和自动测试机制。当团队中有人提交了代码，持续集成系统会触发构建，并将构建的信息反馈到开发者，保证了代码的修改能够让所有平台编译无错误。自动测试机制，就是测试开发人员和功能开发人员一起把功能的测试样例加入自动测试系统中。自动测试系统会按计划在较短的时间段内回归测试，如果测试不通过，会有相关的信息反馈到功能开发者。于此，发出了**异曲同工**之妙的感叹。

## 小结
> 工程项目在Android Studio无法调试C/C++本地代码  
可以修改app.iml的配置属性上治标；也可以从版本控制器里移除app.iml上治本。
> bug处理时机与处理方式和处理代价的关系  
神医扁鹊曾言，大哥医术最高，二哥次之，扁鹊最后。大哥，医人于病前，二哥，医人于病时，扁鹊，医人于病危。扁鹊三兄弟的哲理故事，说明了越早处理越容易，也不用上动刀大手术。《游戏改变世界》提到游戏上瘾的一大特性是：反馈。引入了持续构建和自动测试体系，就能做到二哥的水平，治“虫”于“虫”幼时。

## 附注：另一番折腾
处理 bug 过程在 PC 上的夜神模拟器上工作的，为了避免一次编译多个不同体系下的apk包，就添加了 abiFilters: armeabi-v7a 。然而夜神模拟器则是 X86_64 体系的。

## 参考
* [android studio c/c++调试时无法在c/c++文件中左键打断点-CSDN论坛](https://bbs.csdn.net/topics/392217263?page=1)
* [Android Studio 无法调试 C/C++ 程序 - 死亡叹息的博客 - CSDN博客](https://blog.csdn.net/a40850273/article/details/81699037)
* [android studio调试c/c++代码 - 简书](https://www.jianshu.com/p/45d5ff676db8)
* ✳️ [Debug your app  |  Android Developers](https://developer.android.google.cn/studio/debug/)
* [Android Studio c++ 无法单步调试的系列坑 - 简书](https://www.jianshu.com/p/7f80be68f99b)
* ✳️[Android studio 无法断点调试C问题 - 简书](https://www.jianshu.com/p/d2c8f3caef74)
* ✳️ [intellij idea - What are iml files in Android Studio? - Stack Overflow](https://stackoverflow.com/questions/30737082/what-are-iml-files-in-android-studio)
