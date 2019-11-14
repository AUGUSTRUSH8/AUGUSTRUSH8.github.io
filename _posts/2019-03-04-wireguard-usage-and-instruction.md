---
layout: post
title: '世界那么大我想去看看之WireGuard'
tags: [code]
---

### Ubuntu一键脚本安装WireGuard

Ubuntu的wireguard一键脚本来了，比CentOS版简单，安装快捷，不需要升级内核。

#### 脚本介绍

1、适用于ubuntu版本>=14.04。

2、目前在搬瓦工/vultr的14.04/16.04/18.04版本测试通过。

4、VPS建议Vultr

#### 服务端搭建

连接VPS，使用下面命令

```bash
wget https://raw.githubusercontent.com/atrandys/wireguard/master/wireguard_install_ubuntu.sh && chmod +x wireguard_install_ubuntu.sh && ./wireguard_install_ubuntu.sh
```

选择1 安装wireguard

![](http://image.augustrush8.com/images/wireguard1.png){:.center}

等待安装完成，手机可直接扫描，电脑请下载/etc/wireguard/client.conf文件到电脑。

### 客户端

- windows版

下载安装TunSafe，这是一个windows端的第三方客户端，因为官方windows版本的还没开发完成，先用这个软件代替，TunSafe已经开源了，可以放心使用。

> 官网下载：[TunSafe](https://tunsafe.com/download)

直接选择：

>**下载稳定版安装包：https://tunsafe.com/downloads/TunSafe-1.4.exe**

> **下载测试版安装包：https://tunsafe.com/downloads/TunSafe-1.5-rc1.exe**

打开TunSafe，点击file，选择import file，选择第5步下载的client.conf文件，导入到软件中。

![](http://image.augustrush8.com/images/wireguard2.png){:.center}

导入后会自动连接，连接成功后，所有流量都会被代理，也就是全局代理。

- 安卓版版

1、去Google Play下载wireguard，目前这个软件在Google Play中是未发布版，也可直接下载下面的f-droid的安装包。

> 安卓版wireguard：[点击下载](https://f-droid.org/repo/com.wireguard.android_439.apk)

2、将软件安装好，并将本教程服务端获取的client.conf文件传输到手机中。打开软件，点击加号，在弹出的页面选择create from file or archive，然后选择保存在手机的conf文件。

> 注意：这里可能会提示错误，原因是没有文件操作权限，去权限管理里给软件勾上存储权限即可。

![](http://image.augustrush8.com/images/wireguard3.png){:.center}

选择文件后如下图所示

![](http://image.augustrush8.com/images/wireguard4.png){:.center}

开启代理即可。