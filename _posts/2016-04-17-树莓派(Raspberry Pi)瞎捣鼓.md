---
layout:     post
title:      树莓派(Raspberry Pi)瞎捣鼓
subtitle:   树莓派(Raspberry Pi)瞎捣鼓
date:       2016-04-17
author:     dushenzhi
header-img: img/post-sample-image.jpg
catalog: true
tags:
    - Raspberry Pi
    - Linux
    - 树莓派
---

本周入手了新玩具Raspberry Pi 3 model B，利用周末稍微玩了一下，感觉体验还行，可以用来当个小私服来用或者用来当电视盒子娱乐用。 
点赞点：支持无线网络和蓝牙哦 
吐槽点：貌似没有电源开关按钮，直接拔电源线… 


# 系统安装
------------------------
官方系统下载地址：https://www.raspberrypi.org/downloads/
可以直接在页面下载官方推荐的`NOOBS`，`NOOBS`提供了多个可供Raspberry Pi使用的系统，它分为离线版和在线版：
离线版：内置`Raspbian`系统
在线版：无内置系统，系统安装需要通过在线下载

我下载的是1.9.0离线版，系统安装过程非常简单，过程如下：
（1）只需将用于Raspberry Pi存储数据的SD卡格式化为FAT32格式，建议购买使用大牌，容量大点（建议4G以上）速度快点的SD卡。本人使用的是SanDisk 32GB（class 10）SD卡。
（2）将下载好的`NOOBS_v1_9_0.zip`文件解压后得到`NOOBS_v1_9_0`文件夹，然后将`NOOBS_v1_9_0`文件夹内的所有文件拷贝到格式化好的SD卡上。
（3）将SD卡插入Raspberry Pi上，接上鼠标、键盘、电源和显示器，Raspberry Pi会启动后自动进入`NOOBS`系统安装界面，后面的步骤只需要选择需要安装的系统，然后傻瓜式的点击下一步就可以了。
本人选择安装了官方的`Raspbian`系统，该系统是一个为Raspberry Pi定制的基于Debian的Linux系统
当然也可以选择推荐的第三系统，如Ubuntu MATE等。
# 系统设置
-----------------
###  **安装中文字体和输入法**
默认的raspbian操作系统是不带中文字库的，所以不能正常显示中文字体（中文显示为乱码），可以用apt来安装开源字库的安装包实现中文的显示。
输入命令:
```
 sudo apt-get install ttf-wqy-zenhei
```
这条命令安装的是文泉驿的正黑体。
 
```
sudo apt-get install ttf-wqy-microhei
```
 这条命令安装的是文泉驿的微米黑体。

安装ibus 输入法引擎 和 ibus 拼音输入法：
```
 sudo apt-get install ibus ibus-pinyin
```

### **系统语言、时区和字符集**
（1）图像化界面可以进行如下操作：
![这里写图片描述](http://img.blog.csdn.net/20160416230429813)
(注：图片是汉化后截的，汉化前类同...)
![这里写图片描述](http://img.blog.csdn.net/20160416230534024)
系统语言、国家和字符集信息可以设置为上图中所示，建议在图中Interface tab下开启ssh功能方便远程连接设备。
（2）也可以用命令行下的设置工具进行设置，命令为：`sudo raspi-config`

`reboot`，然后可以正常的显示中文和愉快的输入汉字了....

参考：http://www.linuxidc.com/Linux/2013-04/82805.htm

### **远程桌面**
如果你是使用图形化界面系统建议采用VNC进行远程连接设备，你只需要在Raspberry Pi上安装VNC Server，然后就可以在其他PC机上用VNC Viewer进行连接，避免每次都需要给Raspberry Pi外接显示器。

Raspberry Pi上安装VNC Server命令如下：
```
sudo apt-get install tightvncserver
vncpasswd #设置vnc密码
vncserver #启动vncserver服务，可以加入开启自启避免每次手动启动
```

VNC Viewer官网提供多个平台的支持，下载地址：http://www.realvnc.com/download/viewer/

更多关于VNC使用可以在网上查阅资料...

# 安装chromium浏览器
-----------------------------
内置的浏览器不是很好用，个人还是比较习惯用chrome系列浏览器，raspbian系统默认源无法通过apt-get直接安装chromium浏览器，需要添加第三方源，具体操作如下：

```
wget -qO - http://bintray.com/user/downloadSubjectPublicKey?username=bintray | sudo apt-key add -
echo "deb http://dl.bintray.com/kusti8/chromium-rpi jessie main" | sudo tee -a /etc/apt/sources.list
sudo apt-get update
sudo apt-get install chromium-browser rpi-youtube -y
```

参考：https://www.raspberrypi.org/forums/viewtopic.php?f=63&t=121195

# 安装flash插件
-----------------------
网上找了一些教程很多都是12.xx的版本，不是很好使，浏览器老是奔溃，找了个较新的21.xx版本，亲测可用...
flash插件下载地址：
http://www.mediafire.com/download/0i6v4jdq2y2qved/flash21.tar.xz
已上传的网盘地址：https://pan.baidu.com/s/1jHP2AkI
解压下载到的文件：
```
xz -d flash21.tar.xz
tar xvf flash21.tar
cd pepper/
sudo cp *.so /usr/lib/chromium-browser/plugins/
```
编辑文件，将CHROMIUM_FLAGS设置为如下形式：
```
sudo vim /etc/chromium-browser/default
```

    CHROMIUM_FLAGS="--ppapi-flash-path=/usr/lib/chromium-browser/plugins/libpepflashplayer.so --ppapi-flash-version=21.0.0.182-r1 -password-store=detect -user-data-dir"

重启浏览器，然后可以愉快的在线追剧了有木有...

参考：
https://www.raspberrypi.org/forums/viewtopic.php?t=99202
https://www.raspberrypi.org/forums/viewtopic.php?f=66&t=99202&p=936916#p936916

# 问题
### Omxplayer播放器通过HDMI接口外接电视无声音

通过如下命令播放视频：
```
omxplayer -p -o hdmi xxxx.mkv
```
HDMI接口外接电视没有声音输出，3.5cm音频接口接耳机有声音输出，解决办法：
编辑`/boot/config.txt`文件，设置如下：

    hdmi_drive=2
    
```
reboot  #重启，使配置生效
```




csdn博客文章地址:[https://blog.csdn.net/dushenzhi/article/details/51171207](https://blog.csdn.net/dushenzhi/article/details/51171207)