---
layout: post
title: '利用VPS搭建私人观影平台'
tags: [read]
---

### 前言
以前经常浪迹于互联网观影的各大网站论坛当中，从最开始的百度云观影，发展到种子磁力离线解析，再到直接利用谷歌搜索引擎的抓取能力直接在线观看，再到光顾各种资源采集站，从各种最新的资源流当中收获灵感和观感。其实每种方式都有其独特的好处，比如种子磁力的方式，我们就能看到很多高清的影视资源，而且由于其独特的资源管理存储方式，我们能够更大自由的看到希望看到的东西。但它也有弊端，比如受限于资源的热度，tracker服务器的质量等等。另外，各大在线视频网站或者m3u8类资源采集站也有不小的优势，你可以直接在浏览器当中点播就行，有非常海量的片源供你选择，对网络带宽的要求也并不严格，但也正是以上的这些优点让它也暴露出了明显的缺陷，那就是片源质量不高，而且会被冠以各种广告商的广告词，对于用户的观感体验并不是非常好。
那么，有没有一种稍微geek的方式，让自己这种稍有些技术能力的可以兼顾以上各种方式的优点呢？答案是有的，我们可以借助于PLEX+transmission相结合的方式打造一个你自己的私人观影小蜜。当然是有更好的方式的，比如NAS，但苦于投入对于目前的我来说有些大，故暂不考虑，有朝一日待我钱财充盈再来考虑。缺点？当然也是不少的，比如需要一定的技术基础，需要一定的小额资金投入。

### 搭建流程
流程可主要分为以下四步
>- 拥有自己的VPS
>- 安装配置PLEX
>- 安装配置transmission
>- 自己寻找torrent观影

### 拥有自己的VPS
VPS的功用非常多，有拿来当网盘的、有做游戏加速器的、有做私服的等等，不一而足，能解决很多配置部署的烦恼，推荐把自己的ladder VPS充分利用起来，让它更好的为你服务。我自用的是Aliyun的轻量应用服务器，CentOS7操作系统，1核，2G内存，40G SSD硬盘存储，5Mbps限制峰值带宽。带宽其实很受影响，就目前的需求而言，更主要的是希望带宽能够更大一点，但先将就一下吧，看看效果就知道。主要是便宜。

### 安装配置Plex
- 简单介绍Plex
>PLEX是一款媒体播放器， 有强大的索引、解码功能。如果你上传电影，它可以帮你自动匹配海报、电影介绍等信息；如果你上传的是音乐，它可以匹配专辑封面等。 最重要的是，PLEX一个跨平台的软件，除了各大平台的客户端，它还有网页客户端，打开网页就能播放VPS里面的电影。PLEX还提供付费服务，具体功能可以到PLEX官网查看。

- 安装
```shell
yum -y update && yum -y install wget
wget https://downloads.plex.tv/plex-media-server-new/1.18.2.2058-e67a4e892/redhat/plexmediaserver-1.18.2.2058-e67a4e892.x86_64.rpm
yum install plexmediaserver-1.18.2.2058-e67a4e892.x86_64.rpm
```

以上获取的软件源是按照我的系统版本进行选择的，实际在安装时注意去官网取最新的下载地址，官网地址：`downloads.plex.tv`。
安装完成之后，就只需要开启系统服务就好了。
```shell
systemctl start plexmediaserver #开启服务
systemctl enable plexmediaserver #开机启动
```
可以使用`systemctl status plexmediaserver`查看服务启动状态。看到状态为Active就好

- 配置

1. 打开putty，在connection – S*S*H – Tunnels下设置source port 8888， destination 127.0.0.1:32400，然后点击Add。
2. 点击putty登录，然后打开浏览器，访问http://127.0.0.1:8888/web，这时你就可以看到Plex的Web界面了。
3. 注册一个Plex账号登录，这时Plex会先让你设置一个服务器别名。
4. 接下来就是让你添加自己的媒体库，我们可以选择跳过。
5. 设置完成后，点击进入到Plex服务器主界面，你可以看到已经成功连接上我们的Plex服务器了，有状态、设置、播放列表、频道等

### 添加媒体库

通过上面的设置，现在我们不再需要使用Putty了，直接使用浏览器打开你的Plex服务器IP或者域名地址通过http://IP:32400访问plex了。

紧接着你就可以添加各种不同的媒体文件了，如音乐、电影、图片等等，选定类型以后直接按照指引添加媒体库目录就好了，只需要选定文件夹，具体的文件Plex会自动去扫描添加，最后就只需要在UI界面点击扫描文件就好。

### 功能说明

- 音乐
  - 音乐列表
  - 循环方式选择
  - 暂停
  - 评分
  - 下载
- 图片
  - 幻灯片播放
- 视频
  - 清晰度选择

对于移动端的观影，Plex还提供了对应的供下载使用，界面非常漂亮。

### 安装Transmission

连接SSH，命令行界面输入下面内容

```shell
wget http://github.itzmx.com/1265578519/transmission/master/2.84/transmissionbt.sh -O transmissionbt.sh;sh transmissionbt.sh 
```

执行完成后transmission就安装到了你的服务器上，使用浏览器访问 http://ip:9091，默认账号密码都是 itzmx.com。

这样就完事了！如果需要修改账号、密码和端口等信息的，输入下面的命令行，打开配置文件

```shell
service transmissiond stop
vi /home/transmission/.config/transmission/settings.json
```

主要字段说明

```json
rpc-username 帐号
rpc-password 密码
rpc-port 端口
rpc-authentication-required 是否开启使用账号密码加密访问
```

修改完信息之后，输入：`service transmissiond restart`，重启transmissiond生效

其使用同迅雷差不多，这里不作过多说明。

### 补充说明

实际播放视频当中，发现有些视频格式的解码支持并不非常完全，而我的Plex部署在Centos上面，故我采取了下面的解决方案：

- 安装ffmpeg

首先，`vi ffmpeg.sh`

然后将以下脚本粘贴进去：

```shell
#!/bin/sh
sudo yum -y install epel-release
sudo rpm --import http://li.nux.ro/download/nux/RPM-GPG-KEY-nux.ro
sudo rpm -Uvh http://li.nux.ro/download/nux/dextop/el7/x86_64/nux-dextop-release-0-1.el7.nux.noarch.rpm
yum repolist
sudo  yum -y install ffmpeg ffmpeg-devel
ffmpeg -version
```

执行脚本，`sh ./ffmpeg.sh`，等脚本执行结束就安装完毕了。

接下来，如果需要将flv格式转换为mp4格式，就可以执行类似于下面这样命令

```shell
ffmpeg -i v.flv -y -vcodec copy -acodec copy v.mp4
```

将以上对应的v改为视频名称就好。

### 总结

整体原理可理解为：使用transmission下载资源，PLEX负责展示播放资源文件，利用PLEX的跨平台功能，手机电脑都可以在线播放。

Plex是一整套完整的解决方案，采用Server + Client的形式，Server端用于管理各种媒体（电影，电视剧，照片，音乐、有声小说），Client端用于播放（有Mac，PC，iOS，Android，XBox，PS，各种TV，树莓派等）。需要注意的是PLEX的客户端需要付费才能解决多设备限制。

Plex对播放设备的性能没要求，如果你打算搭建一个Plex Media Server需要注意选择一个好点的服务器。如果影音资料多的话，Plex需要调用大量的CPU来处理转码和输出，所以你的服务器应该在CPU和带宽方面强化配置。

### 参考文章

https://www.kite1874.com/index.php/vps/privatecinema.html 

https://www.howtoing.com/centos-plex-media-server （安装Plex参考）

https://wzfou.com/plex

https://post.smzdm.com/p/a83dm5pl/ （未来可通过它改进为NAS配置）





