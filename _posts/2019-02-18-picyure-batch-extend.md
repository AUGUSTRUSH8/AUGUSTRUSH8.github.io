---
layout: post
title: '图像批量增强脚本'
tags: [code]
---

废话不多说，代码如下，有其他增强需求直接扩展就行：
```python
#-*- coding: UTF-8 -*-   
import os
from PIL import Image
from PIL import ImageEnhance
 
#原始图像
def ImageAugument():
    path = "E:/business/recognition/InvoiceClassification/Keras-image-classifer-framework/traffic-sign-code/traffic-sign/traffic-sign/train/00003" #文件夹目录
    files= os.listdir(path) #得到文件夹下的所有文件名称
    #遍历文件夹
    prefix=path+'/'
    for file in files: 
        
        image = Image.open(prefix+file)
        #image.show()
         
        #亮度增强
        enh_bri = ImageEnhance.Brightness(image)
        brightness = 1.5
        image_brightened = enh_bri.enhance(brightness)
        image_brightened.save(prefix+file[0:6]+'lightup'+'.jpg')
         
        #色度增强
        enh_col = ImageEnhance.Color(image)
        color = 1.5
        image_colored = enh_col.enhance(color)
        image_colored.save(prefix+file[0:6]+'colorup'+'.jpg')
         
        #对比度增强
        enh_con = ImageEnhance.Contrast(image)
        contrast = 1.5
        image_contrasted = enh_con.enhance(contrast)
        image_contrasted.save(prefix+file[0:6]+'contrastup'+'.jpg')
         
        #锐度增强
        enh_sha = ImageEnhance.Sharpness(image)
        sharpness = 3.0
        image_sharped = enh_sha.enhance(sharpness)
        image_sharped.save(prefix+file[0:6]+'moreSharp'+'.jpg')

if __name__ == '__main__':
    ImageAugument()
```
##效果
![image.png](https://upload-images.jianshu.io/upload_images/10780978-3f2679db1e2b6794.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240){:.center}
![image.png](https://upload-images.jianshu.io/upload_images/10780978-45c85b1c3354c590.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240){:.center}