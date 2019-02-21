---
layout: post
title: 'seetaface人脸识别'
tags: [code]
---

先引用一段描述性的文字：

> SeetaFace开源库由FaceDetection、FaceAlignment、FaceIdentification三部分组成。FaceDetection是在一副图片中检测出人脸区域，以一个方形区域表示。FaceAlignment利用FaceDetection中检测出的方框，进一步检测出人脸的5个关键点（两眼中心，鼻尖和两个嘴角）。最后，FaceIdentification利用FaceAlignment中检测出的关键点，提取出单个人脸的特征。

从中可以看出这个开源库的模块功能和特性，下面就简单说一下这个开源模型实践当中的一些坑。



**一点拓展**

最近接触到各种face detection的模型，也对各种方法和模型的优劣有了自己的认识，其中**这个模型**实现简单(但Windows上却又不少麻烦的地方，主要体现在dlib库的安装上面，有兴趣的同学可以参考我之前[安装dlib的文章](https://www.jianshu.com/p/772ccb948f1a))，效果也非常不错，其模型介绍有这样几句话：

> - The world's simplest facial recognition api for Python and the command line
> - The model has an accuracy of 99.38% on the[Labeled Faces in the Wild](https://link.jianshu.com/?t=http%3A%2F%2Fvis-www.cs.umass.edu%2Flfw%2F)benchmark.

这个模型使用起来是简单，但我还没解剖它的实现方法和原理，抽时间去看看。



好了，下面进入正题，seetaface是由中科院山世光团队研究开源的一个模型，据介绍其准确率也能达到97.1%，但很多其他实践的同学总结了这样几条结论：

>检测部分，速度有点慢，误检率小，漏检率稍微大
>对齐部分，还是可以的，但是效果肯定不会有SDM好
>识别部分，按我自己的测试结果，对于变化比较大的2个同样的人，0.7的徘徊分数，不是一个可以实际产品话的算法，个人认为不如DeepID好。
>**其实，这些都不重要，开源精神最值得钦佩。**

个人觉着最后一句话说的很对。
那么，下面我们就自己来实践一下这个开源项目。
网上有很多指导教程或者博客，但大都说的不完全，让很多人有些疑惑，最终跑代码的时候会抛出bug。
其实该开源项目给了非常详尽的指导文档，地址如下: [传送门](https://link.jianshu.com/?t=https%3A%2F%2Fgithub.com%2Fseetaface%2FSeetaFaceEngine%2Fblob%2Fmaster%2FSeetaFace_config.docx)

只要跟着这个指导文档一步一步做就好了，非常详尽。
主要说说注意的地方。我的运行环境是VS2015+opencv3.0+windows，之前的配置基本都没有什么问题，只是到了最后测试的时候由于最新的测试代码有点改动，可能有部分同学不知道最后怎么贴路径，其实稍稍看看测试代码就知道怎么改了，下面贴一个我的测试代码吧，有需要的同学改动一下就好：

```c++
#include <cstdint>
#include <fstream>
#include <iostream>
#include <string>

#include "highgui.hpp"
#include "imgproc.hpp"

#include "face_detection.h"

using namespace std;

int main(int argc, char** argv) {
    const char* img_path = "D:/my_project/hub_project/FaceRecognition_5/MyBuild/Detection/x64/Release/people.jpg";
    seeta::FaceDetection detector("D:/my_project/hub_project/FaceRecognition_5/MyBuild/Detection/x64/Release/seeta_fd_frontal_v1.0.bin");//seeta_fd_frontal_v1.0.bin 文件是作者已经训练好并提供的  
                                                               //设置人脸检测的参数  
    detector.SetMinFaceSize(40);
    detector.SetScoreThresh(2.f);
    detector.SetImagePyramidScaleFactor(0.8f);
    detector.SetWindowStep(4, 4);

    cv::Mat img = cv::imread(img_path, cv::IMREAD_UNCHANGED);
    cv::Mat img_gray;

    if (img.channels() != 1)
        cv::cvtColor(img, img_gray, cv::COLOR_BGR2GRAY);
    else
        img_gray = img;

    seeta::ImageData img_data;
    img_data.data = img_gray.data;//注意这里必须是单通道的图像  
    img_data.width = img_gray.cols;
    img_data.height = img_gray.rows;
    img_data.num_channels = 1;

    long t0 = cv::getTickCount();
    std::vector<seeta::FaceInfo> faces = detector.Detect(img_data);//人脸检测的主要函数  
    long t1 = cv::getTickCount();
    double secs = (t1 - t0) / cv::getTickFrequency();

    cout << "Detections takes " << secs << " seconds " << endl;
#ifdef USE_OPENMP  
    cout << "OpenMP is used." << endl;
#else  
    cout << "OpenMP is not used. " << endl;
#endif  

#ifdef USE_SSE  
    cout << "SSE is used." << endl;
#else  
    cout << "SSE is not used." << endl;
#endif  

    cout << "Image size (wxh): " << img_data.width << "x"
        << img_data.height << endl;

    cv::Rect face_rect;
    int32_t num_face = static_cast<int32_t>(faces.size());

    for (int32_t i = 0; i < num_face; i++) //提取找到的人脸区域  
    {
        face_rect.x = faces[i].bbox.x;
        face_rect.y = faces[i].bbox.y;
        face_rect.width = faces[i].bbox.width;
        face_rect.height = faces[i].bbox.height;

        cv::rectangle(img, face_rect, CV_RGB(0, 0, 255), 4, 8, 0);
    }

    cv::namedWindow("Test", cv::WINDOW_AUTOSIZE);
    cv::imshow("Test", img);
    cv::waitKey(0);
    cv::destroyAllWindows();
}
```

其中要特别注意的一点就是图片和模型的绝对路径要用 **“/”**，Windows里面默认的是“\”,博主在最开始跑代码的时候总是抛出异常，仔细定位分析以后稍改动了以上的内容就成功运行了代码。
我是以简单的face recognition为例跑的代码，随便贴一下我的运行结果吧

![img](https://upload-images.jianshu.io/upload_images/10780978-e65193f81473a483.PNG?imageMogr2/auto-orient/strip%7CimageView2/2/w/801/format/webp)



**后续将会进一步测试face alignment和face identification**