---
layout: post
title: '图像滤波'
tags: [code]
---

**图像其实是一种波,可以用波的算法处理图像**

### 图像VS波?

![bg2017121301.jpg](http://upload-images.jianshu.io/upload_images/10780978-190c80e495129b53.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240){:.center}

以上的lena图来自于花花公子封面,是一张**400X400**的图片,包含160000个像素.
每个像素的颜色，可以用**红、绿、蓝、透明度**四个值描述，大小范围都是**0 ～ 255**，比如黑色是**[0, 0, 0, 255]**，白色是**[255, 255, 255, 255]。**

如果把每一行所有像素（上例是400个）的红、绿、蓝的值，依次画成三条曲线，就得到了下面的图形。
![bg2017121302.png](http://upload-images.jianshu.io/upload_images/10780978-11e54336b4dc5988.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240){:.center}
可以看到，每条曲线都在不停的上下波动。有些区域的波动比较小，有些区域突然出现了大幅波动（比如 54 和 324 这两点）。

对比一下图像就能发现，**曲线波动较大的地方，也是图像出现突变的地方。**
这说明波动与图像是紧密关联的。**图像本质上就是各种色彩波的叠加。**
### 波的频率特性对应图像的什么?

>**色彩剧烈变化的地方，就是图像的高频区域；色彩稳定平滑的地方，就是低频区域。**
### 滤波器:

- 高通
- 低通
  以下两幅图说明了高通和低通的区别:
  ![高通滤波.png](http://upload-images.jianshu.io/upload_images/10780978-cb34eb1c613273f1.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240){:.center}
  ![低通滤波.png](http://upload-images.jianshu.io/upload_images/10780978-32347a3890086ac1.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240){:.center}
### 图像的滤波:

**lowpass**使得图像的高频区域变成低频，即色彩变化剧烈的区域变得平滑，也就是出现**模糊效果**。



![lena_lowpass.jpg](http://upload-images.jianshu.io/upload_images/10780978-04799f85cef8d23b.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240){:.center}
**highpass**正好相反，过滤了低频，只保留那些变化最快速最剧烈的区域，也就是图像里面的物体边缘，所以常用于**边缘识别**。
![lena_highpass.jpg](http://upload-images.jianshu.io/upload_images/10780978-79722e2ed96bace0.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240){:.center}



下面这个[**网址**](http://fellipe.com/demos/lena-js/)

>http://fellipe.com/demos/lena-js/


采用JavaScript运用各个算子对图像进行处理滤波,可以非常鲜明的体会到滤波对图像产生的效果