---
layout: post
title: 'python+opencv检测图片中二维码'
tags: [code]
---

### 缘起
需要检测发票中二维码的位置，以确定图像该怎么旋转，同时也可以为提取二维码信息创造先觉条件！（万恶的需求！）
### 失败的尝试--opencv训练大法
不感兴趣的可跳过不看！
- 参考原文：https://blog.csdn.net/qq_27063119/article/details/79247266
- 解释：原文作者是训练检测舌头。。（蜜汁尴尬），先通过opencv自带的人脸检测cascade分类器进行人脸检测然后叠加训练的舌头分类器完成舌头的检测任务。不多说。
- 我的实践：按照原作者的方法，换个数据集我来尝试一下。
- 正样本：一波处理操作后得到以下样本
  ![image.png](https://upload-images.jianshu.io/upload_images/10780978-a1d32dd477d5f2ca.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240){:.center}
- 负样本：一波骚操作后得到以下样本
  ![image.png](https://upload-images.jianshu.io/upload_images/10780978-e44096898e5d9926.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240){:.center}
- 训练文件夹结构
  ![image.png](https://upload-images.jianshu.io/upload_images/10780978-adf8e6650b0e8011.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240){:.center}
- 测试效果
  ![image.png](https://upload-images.jianshu.io/upload_images/10780978-61084671efa81476.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240){:.center}
- 解释：我解释为训练样本太少，正样本少，负样本也少，原作者说负样本的数量要多于正样本很大一部分，然而我的负样本确实比较少，但我按照模式识别的思想去思考一波，感觉这非常勉强。。。
### 成功的尝试！
#### 第一步：灵感的来源
- 原文链接：https://www.jianshu.com/p/604774f7edb5
- 关键
  ![image.png](https://upload-images.jianshu.io/upload_images/10780978-0aeccefafe1ec544.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240){:.center}
  原作者给出了这么个样式，想一想很明显可以迁移学习的。点开之！
  链接：http://blog.jobbole.com/80448/
#### 第二步：消化以上实现效果的方法
经过一番浏览以后，作者自己就给出了总体的实现思路，如下：
- 计算x方向和y方向上的Scharr梯度幅值表示
- 将x-gradient减去y-gradient来显示条形码区域
- 模糊并二值化图像
- 对二值化图像应用闭运算内核
- 进行系列的腐蚀、膨胀
- 找到图像中的最大轮廓，大概便是条形码
  作者最后的实现效果：
  ![image.png](https://upload-images.jianshu.io/upload_images/10780978-139e43733ccc4f75.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240){:.center}
  ![image.png](https://upload-images.jianshu.io/upload_images/10780978-14d7943d8cd578da.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240){:.center}

可以看出，思路异常清晰！效果也不错，适合自己的需求。
#### 第三步：观察自己的图片
简单处理后是这样的
![image.png](https://upload-images.jianshu.io/upload_images/10780978-6d75abf592faf66b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240){:.center}

- 分析：要识别二维码，这个图片当中颜色区分很明显，所以首先需要把蓝色以外的其他色调给去掉！
#### 第四步：写个小脚本仅显示蓝色调
直接上代码：OnlyBlue.py

```python
import numpy as np
import cv2
import argparse
# 蓝色的范围，不同光照条件下不一样，可灵活调整
lower_blue = np.array([90, 90, 90])
upper_blue = np.array([130, 255, 255])
ap=argparse.ArgumentParser()
ap.add_argument("-i", "--image", required = True, help = "path to the image file")
args = vars(ap.parse_args())

image = cv2.imread(args["image"])
hsv=cv2.cvtColor(image, cv2.COLOR_BGR2HSV)

# 3.inRange()：介于lower/upper之间的为白色，其余黑色
mask = cv2.inRange(hsv, lower_blue, upper_blue)

# 4.只保留原图中的蓝色部分
res = cv2.bitwise_and(image, image, mask=mask)

cv2.imshow('image', image)
cv2.imshow('mask', mask)
cv2.imshow('res', res)
cv2.imwrite('blue.jpg',res)

cv2.waitKey(0)
```

以上代码参考自：[传送门](http://ex2tron.top/2017/12/07/Python-OpenCV%E6%95%99%E7%A8%8B5%EF%BC%9A%E9%A2%9C%E8%89%B2%E7%A9%BA%E9%97%B4%E8%BD%AC%E6%8D%A2/)
也是很好的一篇博客，感兴趣的可以看看

- 解释：由于我这里是比较浅的蓝色调，因此更改了原来代码当中的上下阈值定义部分，如下：

```python
# 蓝色的范围，不同光照条件下不一样，可灵活调整
lower_blue = np.array([90, 90, 90])
upper_blue = np.array([130, 255, 255])
```

- 效果

![img](https://upload-images.jianshu.io/upload_images/10780978-292916417a06242e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/378/format/webp)

这效果我感觉后面已经可以处理了，遂没有再去调阈值参数。

#### 第五步：写检测二维码的程序脚本

直接上代码：

```python
import numpy as np
import argparse
import cv2
ap=argparse.ArgumentParser()
ap.add_argument("-i", "--image", required = True, help = "path to the image file")
args = vars(ap.parse_args())

image = cv2.imread(args["image"])
gray = cv2.cvtColor(image, cv2.COLOR_BGR2GRAY)

gradX = cv2.Sobel(gray, ddepth = cv2.CV_32F, dx = 1, dy = 0, ksize = -1)
gradY = cv2.Sobel(gray, ddepth = cv2.CV_32F, dx = 0, dy = 1, ksize = -1)

gradient = cv2.subtract(gradX, gradY)
gradient = cv2.convertScaleAbs(gradient)

cv2.imshow("gradient",gradient)
#原本没有过滤颜色通道的时候，这个高斯模糊有效，但是如果进行了颜色过滤，不用高斯模糊效果更好
#blurred = cv2.blur(gradient, (9, 9))
(_, thresh) = cv2.threshold(gradient, 225, 255, cv2.THRESH_BINARY)
cv2.imshow("thresh",thresh)
cv2.imwrite('thresh.jpg',thresh)

kernel = cv2.getStructuringElement(cv2.MORPH_RECT, (21, 21))
closed = cv2.morphologyEx(thresh, cv2.MORPH_CLOSE, kernel)
cv2.imshow("closed",closed)
cv2.imwrite('closed.jpg',closed)

closed = cv2.erode(closed, None, iterations = 4)
closed = cv2.dilate(closed, None, iterations = 4)
cv2.imwrite('closed1.jpg',closed)

img,cnts, _ = cv2.findContours(closed.copy(), cv2.RETR_EXTERNAL,cv2.CHAIN_APPROX_SIMPLE)
c = sorted(cnts, key = cv2.contourArea, reverse = True)[0]

rect = cv2.minAreaRect(c)
box = np.int0(cv2.boxPoints(rect))

cv2.drawContours(image, [box], -1, (0, 255, 0), 3)
cv2.imwrite("final.jpg",image)
cv2.imshow("Image", image)

cv2.waitKey(0)
```

#### 关键错误bug解决

原作者文中的代码运行起来有些问题，主要以下两个

- 关于 Python opencv 使用中的 ValueError: too many values to unpack

  - 解决链接：<https://blog.csdn.net/jjddss/article/details/72674704>
  - 关键部分

  ![img](https://upload-images.jianshu.io/upload_images/10780978-a09876db5b609172.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/885/format/webp)

- Why can't use `cv2.cv.BoxPoints` in OpenCV (Python)?

- 解决链接：[传送门](https://stackoverflow.com/questions/48056956/why-cant-use-cv2-cv-boxpoints-in-opencv-python)

- 关键部分

![img](https://upload-images.jianshu.io/upload_images/10780978-b7a4c7821042800c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/736/format/webp)

### 第六步：测试效果

![img](https://upload-images.jianshu.io/upload_images/10780978-7fcd721f55e444a8.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/380/format/webp)

效果很成功！