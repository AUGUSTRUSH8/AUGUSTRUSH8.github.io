---
layout: post
title: 'pix2pix图像超分辨率'
tags: [code]
---

### 目标
训练一个pix2pix模型，把模糊的图片换成清晰的图片
### 准备
租了一个极客云上面的GPU服务器，GTX1070，速度还行
### 数据集准备
数据集是“图片对”的形式，一个图片包含两张图片，一张是清晰的图片，一张是模糊的图片。
#### 我的原数据集
我用的kaggle上面的花花数据：https://www.kaggle.com/alxmamaev/flowers-recognition
#### 去除错误数据
```python
import tensorflow as tf
from glob import glob
import os
import argparse
import logging
from PIL import Image
import traceback

def glob_all(dir_path):
    pic_list = glob(os.path.join(dir_path, '*.jpg'))
    inside = os.listdir(dir_path)
    for dir_name in inside:
        if os.path.isdir(os.path.join(dir_path, dir_name)):
            pic_list.extend(glob_all(os.path.join(dir_path, dir_name)))
    return pic_list

def parse_args():
    parser = argparse.ArgumentParser()
    parser.add_argument('-p', '--dir-path', default='data/')
    return parser.parse_args()

if __name__ == '__main__':
    logging.basicConfig(level=logging.INFO)
    args = parse_args()
    all_pic_list = glob_all(args.dir_path)
    for i, img_path in enumerate(all_pic_list):
        try:
            sess = tf.Session()
            with open(img_path, 'rb') as f:
                img_byte = f.read()
                img = tf.image.decode_jpeg(img_byte)
                data = sess.run(img)
                if data.shape[2] != 3:
                    print(data.shape)
                    raise Exception
            tf.reset_default_graph()
            img = Image.open(img_path)
        except Exception:
            logging.warning('%s has broken. Delete it.' % img_path)
            os.remove(img_path)
        if (i + 1) % 1000 == 0:
            logging.info('Processing %d / %d.' % (i + 1, len(all_pic_list)))
```
运行指令类似如下：
```bash
python delete_broken_img.py -p 文件目录
```
#### 图像裁剪到统一大小
主要两个文件：
process.py:

```python
from __future__ import absolute_import
from __future__ import division
from __future__ import print_function


import argparse
import os
import tempfile
import subprocess
import tensorflow as tf
import numpy as np
import tfimage as im
import threading
import time
import multiprocessing

edge_pool = None


parser = argparse.ArgumentParser()
parser.add_argument("--input_dir", required=True, help="path to folder containing images")
parser.add_argument("--output_dir", required=True, help="output path")
parser.add_argument("--operation", required=True, choices=["grayscale", "resize", "blank", "combine", "edges", "blur"])
parser.add_argument("--workers", type=int, default=1, help="number of workers")
# resize
parser.add_argument("--pad", action="store_true", help="pad instead of crop for resize operation")
parser.add_argument("--size", type=int, default=256, help="size to use for resize operation")
# combine
parser.add_argument("--b_dir", type=str, help="path to folder containing B images for combine operation")
a = parser.parse_args()


def resize(src):
    height, width, _ = src.shape
    dst = src
    if height != width:
        if a.pad:
            size = max(height, width)
            # pad to correct ratio
            oh = (size - height) // 2
            ow = (size - width) // 2
            dst = im.pad(image=dst, offset_height=oh, offset_width=ow, target_height=size, target_width=size)
        else:
            # crop to correct ratio
            size = min(height, width)
            oh = (height - size) // 2
            ow = (width - size) // 2
            dst = im.crop(image=dst, offset_height=oh, offset_width=ow, target_height=size, target_width=size)

    assert(dst.shape[0] == dst.shape[1])

    size, _, _ = dst.shape
    if size > a.size:
        dst = im.downscale(images=dst, size=[a.size, a.size])
    elif size < a.size:
        dst = im.upscale(images=dst, size=[a.size, a.size])
    return dst


def blank(src):
    height, width, _ = src.shape
    if height != width:
        raise Exception("non-square image")

    image_size = width
    size = int(image_size * 0.3)
    offset = int(image_size / 2 - size / 2)

    dst = src
    dst[offset:offset + size, offset:offset + size, :] = np.ones([size, size, 3])
    return dst


def combine(src, src_path):
    if a.b_dir is None:
        raise Exception("missing b_dir")

    # find corresponding file in b_dir, could have a different extension
    basename, _ = os.path.splitext(os.path.basename(src_path))
    for ext in [".png", ".jpg"]:
        sibling_path = os.path.join(a.b_dir, basename + ext)
        if os.path.exists(sibling_path):
            sibling = im.load(sibling_path)
            break
    else:
        raise Exception("could not find sibling image for " + src_path)

    # make sure that dimensions are correct
    height, width, _ = src.shape
    if height != sibling.shape[0] or width != sibling.shape[1]:
        raise Exception("differing sizes")

    # convert both images to RGB if necessary
    if src.shape[2] == 1:
        src = im.grayscale_to_rgb(images=src)

    if sibling.shape[2] == 1:
        sibling = im.grayscale_to_rgb(images=sibling)

    # remove alpha channel
    if src.shape[2] == 4:
        src = src[:, :, :3]

    if sibling.shape[2] == 4:
        sibling = sibling[:, :, :3]

    return np.concatenate([src, sibling], axis=1)


def grayscale(src):
    return im.grayscale_to_rgb(images=im.rgb_to_grayscale(images=src))


def blur(src, scale=4):
    height, width, _ = src.shape
    height_down = height // scale
    width_down = width // scale
    dst = im.downscale(images=src, size=[height_down, width_down])
    dst = im.upscale(images=dst, size=[height, width])
    return dst

net = None


def run_caffe(src):
    # lazy load caffe and create net
    global net
    if net is None:
        # don't require caffe unless we are doing edge detection
        os.environ["GLOG_minloglevel"] = "2"  # disable logging from caffe
        import caffe
        # using this requires using the docker image or assembling a bunch of dependencies
        # and then changing these hardcoded paths
        net = caffe.Net("/opt/caffe/examples/hed/deploy.prototxt", "/opt/caffe/hed_pretrained_bsds.caffemodel", caffe.TEST)

    net.blobs["data"].reshape(1, *src.shape)
    net.blobs["data"].data[...] = src
    net.forward()
    return net.blobs["sigmoid-fuse"].data[0][0, :, :]


def edges(src):
    # based on https://github.com/phillipi/pix2pix/blob/master/scripts/edges/batch_hed.py
    # and https://github.com/phillipi/pix2pix/blob/master/scripts/edges/PostprocessHED.m
    import scipy.io
    src = src * 255
    border = 128  # put a padding around images since edge detection seems to detect edge of image
    src = src[:, :, :3]  # remove alpha channel if present
    src = np.pad(src, ((border, border), (border, border), (0, 0)), "reflect")
    src = src[:, :, ::-1]
    src -= np.array((104.00698793, 116.66876762, 122.67891434))
    src = src.transpose((2, 0, 1))

    # [height, width, channels] => [batch, channel, height, width]
    fuse = edge_pool.apply(run_caffe, [src])
    fuse = fuse[border:-border, border:-border]

    with tempfile.NamedTemporaryFile(suffix=".png") as png_file, tempfile.NamedTemporaryFile(suffix=".mat") as mat_file:
        scipy.io.savemat(mat_file.name, {"input": fuse})

        octave_code = r"""
E = 1-load(input_path).input;
E = imresize(E, [image_width,image_width]);
E = 1 - E;
E = single(E);
[Ox, Oy] = gradient(convTri(E, 4), 1);
[Oxx, ~] = gradient(Ox, 1);
[Oxy, Oyy] = gradient(Oy, 1);
O = mod(atan(Oyy .* sign(-Oxy) ./ (Oxx + 1e-5)), pi);
E = edgesNmsMex(E, O, 1, 5, 1.01, 1);
E = double(E >= max(eps, threshold));
E = bwmorph(E, 'thin', inf);
E = bwareaopen(E, small_edge);
E = 1 - E;
E = uint8(E * 255);
imwrite(E, output_path);
"""

        config = dict(
            input_path="'%s'" % mat_file.name,
            output_path="'%s'" % png_file.name,
            image_width=256,
            threshold=25.0 / 255.0,
            small_edge=5,
        )

        args = ["octave"]
        for k, v in config.items():
            args.extend(["--eval", "%s=%s;" % (k, v)])

        args.extend(["--eval", octave_code])
        try:
            subprocess.check_output(args, stderr=subprocess.STDOUT)
        except subprocess.CalledProcessError as e:
            print("octave failed")
            print("returncode:", e.returncode)
            print("output:", e.output)
            raise
        return im.load(png_file.name)


def process(src_path, dst_path):
    src = im.load(src_path)

    if a.operation == "grayscale":
        dst = grayscale(src)
    elif a.operation == "resize":
        dst = resize(src)
    elif a.operation == "blank":
        dst = blank(src)
    elif a.operation == "combine":
        dst = combine(src, src_path)
    elif a.operation == "edges":
        dst = edges(src)
    elif a.operation == "blur":
        dst = blur(src)
    else:
        raise Exception("invalid operation")

    im.save(dst, dst_path)


complete_lock = threading.Lock()
start = None
num_complete = 0
total = 0


def complete():
    global num_complete, rate, last_complete

    with complete_lock:
        num_complete += 1
        now = time.time()
        elapsed = now - start
        rate = num_complete / elapsed
        if rate > 0:
            remaining = (total - num_complete) / rate
        else:
            remaining = 0

        print("%d/%d complete  %0.2f images/sec  %dm%ds elapsed  %dm%ds remaining" % (num_complete, total, rate, elapsed // 60, elapsed % 60, remaining // 60, remaining % 60))

        last_complete = now


def main():
    if not os.path.exists(a.output_dir):
        os.makedirs(a.output_dir)

    src_paths = []
    dst_paths = []

    skipped = 0
    for src_path in im.find(a.input_dir):
        name, _ = os.path.splitext(os.path.basename(src_path))
        dst_path = os.path.join(a.output_dir, name + ".png")
        if os.path.exists(dst_path):
            skipped += 1
        else:
            src_paths.append(src_path)
            dst_paths.append(dst_path)

    print("skipping %d files that already exist" % skipped)

    global total
    total = len(src_paths)

    print("processing %d files" % total)

    global start
    start = time.time()

    if a.operation == "edges":
        # use a multiprocessing pool for this operation so it can use multiple CPUs
        # create the pool before we launch processing threads
        global edge_pool
        edge_pool = multiprocessing.Pool(a.workers)

    if a.workers == 1:
        with tf.Session() as sess:
            for src_path, dst_path in zip(src_paths, dst_paths):
                process(src_path, dst_path)
                complete()
    else:
        queue = tf.train.input_producer(zip(src_paths, dst_paths), shuffle=False, num_epochs=1)
        dequeue_op = queue.dequeue()

        def worker(coord):
            with sess.as_default():
                while not coord.should_stop():
                    try:
                        src_path, dst_path = sess.run(dequeue_op)
                    except tf.errors.OutOfRangeError:
                        coord.request_stop()
                        break

                    process(src_path, dst_path)
                    complete()

        # init epoch counter for the queue
        local_init_op = tf.local_variables_initializer()
        with tf.Session() as sess:
            sess.run(local_init_op)

            coord = tf.train.Coordinator()
            threads = tf.train.start_queue_runners(coord=coord)
            for i in range(a.workers):
                t = threading.Thread(target=worker, args=(coord,))
                t.start()
                threads.append(t)

            try:
                coord.join(threads)
            except KeyboardInterrupt:
                coord.request_stop()
                coord.join(threads)

main()

```
tfimage.py：
```python
from __future__ import absolute_import
from __future__ import division
from __future__ import print_function

import tensorflow as tf
import os


def create_op(func, **placeholders):
    op = func(**placeholders)

    def f(**kwargs):
        feed_dict = {}
        for argname, argvalue in kwargs.items():
            placeholder = placeholders[argname]
            feed_dict[placeholder] = argvalue
        return tf.get_default_session().run(op, feed_dict=feed_dict)

    return f

downscale = create_op(
    func=tf.image.resize_images,
    images=tf.placeholder(tf.float32, [None, None, None]),
    size=tf.placeholder(tf.int32, [2]),
    method=tf.image.ResizeMethod.AREA,
)

upscale = create_op(
    func=tf.image.resize_images,
    images=tf.placeholder(tf.float32, [None, None, None]),
    size=tf.placeholder(tf.int32, [2]),
    method=tf.image.ResizeMethod.BICUBIC,
)

decode_jpeg = create_op(
    func=tf.image.decode_jpeg,
    contents=tf.placeholder(tf.string),
)

decode_png = create_op(
    func=tf.image.decode_png,
    contents=tf.placeholder(tf.string),
)

rgb_to_grayscale = create_op(
    func=tf.image.rgb_to_grayscale,
    images=tf.placeholder(tf.float32),
)

grayscale_to_rgb = create_op(
    func=tf.image.grayscale_to_rgb,
    images=tf.placeholder(tf.float32),
)

encode_jpeg = create_op(
    func=tf.image.encode_jpeg,
    image=tf.placeholder(tf.uint8),
)

encode_png = create_op(
    func=tf.image.encode_png,
    image=tf.placeholder(tf.uint8),
)

crop = create_op(
    func=tf.image.crop_to_bounding_box,
    image=tf.placeholder(tf.float32),
    offset_height=tf.placeholder(tf.int32, []),
    offset_width=tf.placeholder(tf.int32, []),
    target_height=tf.placeholder(tf.int32, []),
    target_width=tf.placeholder(tf.int32, []),
)

pad = create_op(
    func=tf.image.pad_to_bounding_box,
    image=tf.placeholder(tf.float32),
    offset_height=tf.placeholder(tf.int32, []),
    offset_width=tf.placeholder(tf.int32, []),
    target_height=tf.placeholder(tf.int32, []),
    target_width=tf.placeholder(tf.int32, []),
)

to_uint8 = create_op(
    func=tf.image.convert_image_dtype,
    image=tf.placeholder(tf.float32),
    dtype=tf.uint8,
    saturate=True,
)

to_float32 = create_op(
    func=tf.image.convert_image_dtype,
    image=tf.placeholder(tf.uint8),
    dtype=tf.float32,
)


def load(path):
    with open(path, "rb") as f:
        contents = f.read()
        
    _, ext = os.path.splitext(path.lower())

    if ext == ".jpg":
        image = decode_jpeg(contents=contents)
    elif ext == ".png":
        image = decode_png(contents=contents)
    else:
        raise Exception("invalid image suffix")

    return to_float32(image=image)


def find(d):
    result = []
    for filename in os.listdir(d):
        _, ext = os.path.splitext(filename.lower())
        if ext == ".jpg" or ext == ".png":
            result.append(os.path.join(d, filename))
    result.sort()
    return result


def save(image, path, replace=False):
    _, ext = os.path.splitext(path.lower())
    image = to_uint8(image=image)
    if ext == ".jpg":
        encoded = encode_jpeg(image=image)
    elif ext == ".png":
        encoded = encode_png(image=image)
    else:
        raise Exception("invalid image suffix")

    dirname = os.path.dirname(path)
    if dirname != "" and not os.path.exists(dirname):
        os.makedirs(dirname)

    if os.path.exists(path):
        if replace:
            os.remove(path)
        else:
            raise Exception("file already exists at " + path)

    with open(path, "wb") as f:
        f.write(encoded)

```
运行指令类似下面：
```bash
python process.py --input_dir 上步处理完的文件目录 --operation resize --output_dir 自己定义一个输出文件夹
```
#### 制作对应要求的图片对
代码在：https://github.com/hzy46/Deep-Learning-21-Examples/blob/master/chapter_10/
第十章的代码，对应的处理代码在chapter10/pix2pix-tensorflow/tools下，需要的两个处理脚本与上面的两个脚本同名。下载下来放到对应文件夹就好。
- 模糊处理命令
```bash
python process.py --operation blur --input_dir resize后的文件目录 --output_dir 自定义一个输出文件夹
```
- 原始图片和模糊图片合并在一起命令
```bash
python process.py --input_dir resize后的文件夹 --b_dir 上面模糊操作后的输出文件夹 --operation combine --output_dir 自定义输出文件夹
```
- 分为训练集和测试集（split.py同样在之前的GitHub地址）
```bash
python split.py --dir 上面的合并后输出文件夹
```
#### 最后的生成结果类似下面
![image.png](https://upload-images.jianshu.io/upload_images/10780978-ae1bfb6541e4392e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240){:.center}
![image.png](https://upload-images.jianshu.io/upload_images/10780978-047a775d6d633091.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240){:.center}

### 训练模型（pix2pix.py还是在上面GitHub地址）
```bash
python pix2pix.py --mode train --output_dir 自定义模型输出模型路径 --max_epochs 20 --input_dir 上面输出的训练文件夹 --which_direction BtoA
```
### 模型迭代
由于我把终端给关了，就不截图了，最后云端打包下来是这样的形式：
![image.png](https://upload-images.jianshu.io/upload_images/10780978-3b2dabef3af9175d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240){:.center}

### 测试模型
```bash
python pix2pix.py --mode test --output_dir 自定义输出文件夹 --input_dir 之前生成的验证数据集目录 --checkpoint 之前自定义的模型输出文件夹
```
### 结果展示
![image.png](https://upload-images.jianshu.io/upload_images/10780978-5cdce9bb3cce775d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240){:.center}
左边是模糊的，中间是模型生成的，右边是原图，效果不是特别好，之前看了看我的模型，后面收敛的不是很好，但差不多就这意思了，另外我的数据集也不太好，仅作借鉴