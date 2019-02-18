---
layout: post
title: '神奇的YOLOv2'
tags: [code]
---

很久以前就听说过yolo的过人之处，直到这两天，因为需要做一个物体检测标定位置的活儿，因此不得不就这个方向的研究进行一定的学习和摸索。今天，就让我们一起来见证一下这个YOLO的神奇之处。
##### 介绍一下
>**YOLO核心思想**：从R-CNN到Fast R-CNN一直采用的思路是proposal+分类 （proposal 提供位置信息， 分类提供类别信息）精度已经很高，但是速度还不行。 YOLO提供了另一种更为直接的思路： 直接在输出层回归bounding box的位置和bounding box所属的类别(整张图作为网络的输入，把 Object Detection 的问题转化成一个 Regression 问题)。

>**YOLO的主要特点**：
>- 速度快，能够达到实时的要求。在 Titan X 的 GPU 上 能够达到 45 帧每秒。
>- 使用全图作为 Context 信息，背景错误（把背景错认为物体）比较少。
>- 泛化能力强。

>**网络设计**
>![image](http://upload-images.jianshu.io/upload_images/10780978-50bbcc2f5e5ba1fc.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240){:.center}
>![捕获.PNG](https://upload-images.jianshu.io/upload_images/10780978-beb7971db352589a.PNG?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

以上内容摘自知乎**@晓雷**的笔记，详细内容请看**[原文](https://zhuanlan.zhihu.com/p/24916786)**
再来看看人家官网的描述：
![捕获1.PNG](https://upload-images.jianshu.io/upload_images/10780978-834ada919936c33f.PNG?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240){:.center}
![捕获2.PNG](https://upload-images.jianshu.io/upload_images/10780978-1621f6d2b7642d13.PNG?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240){:.center}
![demo.png](https://upload-images.jianshu.io/upload_images/10780978-86e4c1cb3501e483.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240){:.center}
![demo1.png](https://upload-images.jianshu.io/upload_images/10780978-588f44e75e42b98d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240){:.center}

嗯！非常有极客范的一个网站，蠢蠢欲动！
***
#####看看它有多厉害！
下面是Siraj Raval的一个演示视频，我觉着可以充分说明这个YOLO有多厉害：
>https://youtu.be/4eIBisqx9_g

emmm~需要科学上网。
#####实践之路
我的电脑是Win10 64的，所以我是在Windows上面实现的，以下以我的环境为例进行阐述。而且也是以训练好的模型进行阐述。训练自己的数据集将在之后作进一步的探索。网上也有很多Linux实现的，想尝试的同学也可以搜索实践一下。
- 第一步
  按照文章**[地址](https://ganjiacheng.cn/blog/?p=300)**的描述了解一个大概的流程就好，建议不要按照博主的配置走，因为原作者已经在GitHub上面进行了更新和配置说明，跟着GitHub作者的描述走更加顺利。**[GitHub地址](https://github.com/AlexeyAB/darknet#how-to-use)**
  我的配置流程主要是按照下面的描述走的：
***
### How to compile on Windows:

1. If you have **MSVS 2015, CUDA 9.1, cuDNN 7.0 and OpenCV 3.x** (with paths: `C:\opencv_3.0\opencv\build\include` & `C:\opencv_3.0\opencv\build\x64\vc14\lib`), then start MSVS, open `build\darknet\darknet.sln`, set **x64** and **Release**, and do the: Build -> Build darknet. **NOTE:** If installing OpenCV, use OpenCV 3.4.0 or earlier. This is a bug in OpenCV 3.4.1 in the C API (see [#500](https://github.com/AlexeyAB/darknet/issues/500)).

    1.1\. Find files `opencv_world320.dll` and `opencv_ffmpeg320_64.dll` (or `opencv_world340.dll` and `opencv_ffmpeg340_64.dll`) in `C:\opencv_3.0\opencv\build\x64\vc14\bin` and put it near with `darknet.exe`

    1.2 Check that there are `bin` and `include` folders in the `C:\Program Files\NVIDIA GPU Computing Toolkit\CUDA\v9.1` if aren't, then copy them to this folder from the path where is CUDA installed

    1.3\. To install CUDNN (speedup neural network), do the following:

    *   download and install **cuDNN 7.0 for CUDA 9.1**: [https://developer.nvidia.com/cudnn](https://developer.nvidia.com/cudnn)

    *   add Windows system variable `cudnn` with path to CUDNN: [https://hsto.org/files/a49/3dc/fc4/a493dcfc4bd34a1295fd15e0e2e01f26.jpg](https://hsto.org/files/a49/3dc/fc4/a493dcfc4bd34a1295fd15e0e2e01f26.jpg)

    1.4\. If you want to build **without CUDNN** then: open `\darknet.sln` -> (right click on project) -> properties -> C/C++ -> Preprocessor -> Preprocessor Definitions, and remove this: `CUDNN;`

2. If you have other version of **CUDA (not 9.1)** then open `build\darknet\darknet.vcxproj` by using Notepad, find 2 places with "CUDA 9.1" and change it to your CUDA-version, then do step 1

3. If you **don't have GPU**, but have **MSVS 2015 and OpenCV 3.0** (with paths: `C:\opencv_3.0\opencv\build\include` & `C:\opencv_3.0\opencv\build\x64\vc14\lib`), then start MSVS, open `build\darknet\darknet_no_gpu.sln`, set **x64** and **Release**, and do the: Build -> Build darknet_no_gpu

4. If you have **OpenCV 2.4.13** instead of 3.0 then you should change pathes after `\darknet.sln` is opened

    4.1 (right click on project) -> properties -> C/C++ -> General -> Additional Include Directories:`C:\opencv_2.4.13\opencv\build\include`

    4.2 (right click on project) -> properties -> Linker -> General -> Additional Library Directories: `C:\opencv_2.4.13\opencv\build\x64\vc14\lib`
***
其中需要注意的就是以上**1.1**的描述。基本环境编译没问题以后，就可以接着**[这篇文章](https://ganjiacheng.cn/blog/?p=300)**继续进行了。后面的流程基本没什么问题
#####效果展示
下面展示一些我的测试案例吧
程序初始化：
![process.PNG](https://upload-images.jianshu.io/upload_images/10780978-64af58ddc6fa020e.PNG?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240){:.center}
测试案例：
![test1.PNG](https://upload-images.jianshu.io/upload_images/10780978-eb72b88d80d46486.PNG?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240){:.center}
![test2.PNG](https://upload-images.jianshu.io/upload_images/10780978-9a5ed1f636d725fa.PNG?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240){:.center}

还是很令人激动有木有！
好了，今天介绍到此结束，后续训练自己的数据集！