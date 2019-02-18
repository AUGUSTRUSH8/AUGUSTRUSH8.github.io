---
layout: post
title: '与CPTN（文字识别网络）作斗争的记录'
tags: [code]
---

## CTPN是什么
CTPN结合CNN与LSTM深度网络，CTPN是从Faster R-CNN改进而来，能有效的检测出复杂场景的横向分布的文字，效果如图1，是目前比较好的文字检测算法。详细解释：[传送门](https://zhuanlan.zhihu.com/p/34757009)
![image.png](https://upload-images.jianshu.io/upload_images/10780978-009e5f5c05cdf909.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240){:.center}
说人话：文字识别的前期工作，需要把图片中的文字区域定位出来，然后可以做适当的裁切作进一步的文字识别工作！

## 本次要实现的项目
地址如下（别着急克隆）：
>https://github.com/eragonruan/text-detection-ctpn

![image.png](https://upload-images.jianshu.io/upload_images/10780978-566da9acd2ca82ce.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240){:.center}
## 项目要实现的效果
![image.png](https://upload-images.jianshu.io/upload_images/10780978-5d36504ae0bc8cae.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240){:.center}
![image.png](https://upload-images.jianshu.io/upload_images/10780978-766026deb693123b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240){:.center}

## 第一步--克隆项目
我克隆的是下面这个项目，他有训练好的权重文件：
>https://github.com/GuoYi0/text_detection

## 第二步--开始跑demo
结果碰到了下面这个错误：
>ImportError: cannot import name 'bbox'

回去找issue，果然有人跟我一样，才看半截就开跑
issue地址：https://github.com/eragonruan/text-detection-ctpn/issues/59
下面有人给出了答案：
![image.png](https://upload-images.jianshu.io/upload_images/10780978-d648ee1eedca7b06.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

照着做，再跑一次，但是，，，还是
![image.png](https://upload-images.jianshu.io/upload_images/10780978-9ee6aedf64a9a112.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240){:.center}
another error：

> Invalid argument: ValueError: Buffer dtype mismatch, expected 'int_t' but got 'long long'

再去找解决方案。找到了下面这个：
issue地址：https://github.com/eragonruan/text-detection-ctpn/issues/59
解决方案如下（我只是部分修改，原作者是仅使用CPU执行）
```txt
thanks to the author and [#43](https://github.com/eragonruan/text-detection-ctpn/issues/43) zhao181

my environment is:
windows10 ,
python3.6 ,
tensorflow1.3 ,
vs2015(ps:vs2013 not support python3.6 when compile)

step 1:make some change
change "np.int_t " to "np.intp_t" in line 25 of the file lib\utils\cython_nms.pyx
otherwise appear " ValueError: Buffer dtype mismatch, expected 'int_t' but got 'long long' " in step 6.

step 2:updata c file
execute:cd your_dir\text-detection-ctpn-master\lib\utils
execute:cython bbox.pyx
execute:cython cython_nms.pyx

step 3:builf setup file as setup_new.py
import numpy as np
from distutils.core import setup
from Cython.Build import cythonize
from distutils.extension import Extension
numpy_include = np.get_include()
setup(ext_modules=cythonize("bbox.pyx"),include_dirs=[numpy_include])
setup(ext_modules=cythonize("cython_nms.pyx"),include_dirs=[numpy_include])

step 4:build .pyd file
execute:python setup_new.py install
copy bbox.cp36-win_amd64.pyd and cython_nms.cp36-win_amd64.pyd to your_dir\text-detection-ctpn-master\lib\utils

step 5:make some change
(1) Set "USE_GPU_NMS " in the file \ctpn\text.yml as "False"
(2) Set the "_*C.USE_GPU_NMS" in the file \lib\fast_rcnn\config.py as "False";
(3) Comment out the line "from lib.utils.gpu_nms import gpu_nms" in the file \lib\fast_rcnn\nms_wrapper.py;
(4) Comment out the line "from . import gpu_nms" in the file \lib\utils_*init**.py;
(5) change "base_name = image_name.split('/')[-1]" to "base_name = image_name.split('\')[-1]" in line 24 of the file ctpn\demo.py

step 6:run demo
execute:cd your_dir\text-detection-ctpn-master
execute:python ./ctpn/demo.py

```
关键改动部分
```txt
step 1:make some change
change "np.int_t " to "np.intp_t" in line 25 of the file lib\utils\cython_nms.pyx
otherwise appear " ValueError: Buffer dtype mismatch, expected 'int_t' but got 'long long' " in step 6.
```
我再跑一次测试！
结果，，，
![image.png](https://upload-images.jianshu.io/upload_images/10780978-6f6623873805bb95.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240){:.center}
错误信息如下：

```txt
Loading network VGGnet_test...  Restoring from checkpoints/VGGnet_fast_rcnn_iter_50000.ckpt... done
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
Demo for E:\business\recognition\text_detection\text-detection-ctpn\data\demo\010.png
Traceback (most recent call last):
  File "./ctpn/demo.py", line 101, in <module>
    ctpn(sess, net, im_name)
  File "./ctpn/demo.py", line 61, in ctpn
    draw_boxes(img, image_name, boxes, scale)
  File "./ctpn/demo.py", line 26, in draw_boxes
    with open('data/results/' + 'res_{}.txt'.format(base_name.split('.')[0]), 'w') as f:
OSError: [Errno 22] Invalid argument: 'data/results/res_E:\\business\\recognition\\text_detection\\text-detection-ctpn\\data\\demo\\010.txt'
```
定位这个问题花了我一点时间，各种猜想，最后再回头详细读一下这个错误信息，发现了问题所在，最后一句！参数错误！
回去改源码：
```txt
 base_name = image_name.split('\\')[-1]
    with open('data\\results\\' + 'res_{}.txt'.format(base_name.split('.')[0]), 'w') as f:
```
只想说一句，Windows的**“\”**真是。。。
![image.png](https://upload-images.jianshu.io/upload_images/10780978-703b07913d669a5d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240){:.center}
我再跑！功夫不负有心人，输出结果如下 
![image.png](https://upload-images.jianshu.io/upload_images/10780978-4a160c52fc49953d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
查看输出结果目录
![image.png](https://upload-images.jianshu.io/upload_images/10780978-31abe20089062227.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240){:.center}
接下来就是去研究原理和源码了，这个得放一段落，先把业务完成先。。。
![image.png](https://upload-images.jianshu.io/upload_images/10780978-366f930dad8f4b77.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240){:.center}

再补一句，对这个感兴趣的还可以去研究下下面这个项目，提供了数据集等

>https://github.com/QAlexBall/Faster_RCNN_for_TextDetection