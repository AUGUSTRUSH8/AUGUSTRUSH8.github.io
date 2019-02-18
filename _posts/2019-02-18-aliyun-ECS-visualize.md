---
layout: post
title: '给阿里云主机装个可视化桌面'
tags: [code]
---

基本过程是按照**[这篇博客](https://blog.csdn.net/dk_0228/article/details/54571867)**来的，但期间碰到点小问题--最后一步远程连接桌面连不上，通过搜索，解决了该问题
- 第一步，买一个穷人版主机，这个不多说，有钱淫可以搞个配置好点的，我的一核2G。
- 安装**putty**远程主机连接软件，也没什么好说的
- 通过putty连上远程主机
  ![image.png](https://upload-images.jianshu.io/upload_images/10780978-6e20aaadb06a0e70.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240){:.center}
  ![image.png](https://upload-images.jianshu.io/upload_images/10780978-86df89f621e47da8.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240){:.center}
- 安装vncserver，这个是用来远程连接用的。命令如下 
```shell
apt-get install vnc4server
```
在这一步出错了，apt-get update一下就好了。
- 安装图形界面
```
apt-get install xfce4如果安装不上，就 
apt-get update 
apt-get upgrade更新一下，一般是没有问题的
```
- 启动vncserver,首先我们先运行一下，以生成配置文件 
  **vncserver :1**这时候需要你输入一个8位数的密码，这个密码你需要记住，这个是你以后远程连接要用到的。 
  然后我们再把它kill掉来修改启动文件 
  **vncserver -kill :1**
- 修改vnc的启动文件 
  vi ~/.vnc/xstartup在里面将最后一行注释掉 
  x-window-manager &就是它。在前面加个’#’就注释掉了 
  然后加上我们的界面xfce的相关内容
```
sesion-manager & xfdesktop & xfce4-panel &   
xfce4-menu-plugin &   
xfsettingsd &   
xfconfd &   
xfwm4 &   
```
改完后：
```
#!/bin/sh

# Uncomment the following two lines for normal desktop:
# unset SESSION_MANAGER
# exec /etc/X11/xinit/xinitrc
#xrdb $HOME/.Xresources
#xsettroot -solid grey
#startxfce4&

[ -x /etc/vnc/xstartup ] && exec /etc/vnc/xstartup
[ -r $HOME/.Xresources ] && xrdb $HOME/.Xresources
xsetroot -solid grey
vncconfig -iconic &

x-terminal-emulator -geometry 80x24+10+10 -ls -title "$VNCDESKTOP Desktop" &
#x-window-manager &

sesion-manager & xfdesktop & xfce4-panel &
xfce4-menu-plugin &
xfsettingsd &
xfconfd &
xfwm4 &
```
然后保存退出。先按ESC键，然后输入:wq,最后按回车键就好了。
- 我们再次启动vncserver用来远程连接。 
  vncserver :1 后面的1是后来连接需要的。
- 在你的电脑上安装vncviwer,用来远程连接。
  这里原作者给了链接，但是链接挂了，自己百度下一个就行
  打开刚刚下的那软件，按下面提示输入
  ![image.png](https://upload-images.jianshu.io/upload_images/10780978-4ba184f914a9c86e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240){:.center}
  可我在这里碰到了问题，提示信息如下：
```
time out waiting for a response from the computer
```
因此百度解决之，在阿里的官方文档当中发现了下面这个
![image.png](https://upload-images.jianshu.io/upload_images/10780978-c9ab6f54f21098de.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240){:.center}
附：阿里原文地址,看原文比较有效

>https://help.aliyun.com/knowledge_detail/41530.html

从上图中看书信息还是很有效的，需要使用TCP 5901端口，之前几番百度也有修改访问规则配置的文章，所以驾轻就熟
![image.png](https://upload-images.jianshu.io/upload_images/10780978-e9a24e48596fd68b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240){:.center}
点那个添加安全组规则就好，修改端口！

- 再次输入--**你的公网IP:1**，下面的东西就出来了，enjoy！
  ![image.png](https://upload-images.jianshu.io/upload_images/10780978-c5fcd369a2503c75.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240){:.center}