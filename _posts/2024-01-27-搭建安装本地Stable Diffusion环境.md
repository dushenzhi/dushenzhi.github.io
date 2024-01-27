---
layout:     post
title:      搭建安装本地Stable Diffusion环境
subtitle:   Stable Diffusion入门
date:       2024-01-27
author:     dushenzhi
header-img: img/post-sample-image.jpg
catalog: true
tags:
    - Stable Diffusion
    - Lora
    - AIGC
---

## Windows系统
### 方法一
* 1.下载[sd.webui.zip](https://github.com/AUTOMATIC1111/stable-diffusion-webui/releases/tag/v1.0.0-pre),该压缩包会适时更新到webui最新版本
* 2.本地解压下载的`.zip`压缩文件
* 3.双击解压文件夹下的`update.bat`文件，更新web UI到最新版本
* 4.双击解压文件夹下的`run.bat`文件启动web UI，第一次启动需要下载大量的文件，会比较慢。待所有文件下载和安装完毕，可以看到一条提示消息：`Running on local URL:  http://127.0.0.1:7860`，打开浏览器访问该URL连接即可访问web UI

### 方法二
* 1.安装[Python 3.10.6](https://www.python.org/ftp/python/3.10.6/python-3.10.6-amd64.exe)(请确保勾选了“Add Python 3.10 to PATH”选项)和[Git](https://github.com/git-for-windows/git/releases/download/v2.39.2.windows.1/Git-2.39.2-64-bit.exe)
* 2.打开命令行窗口，执行命令：`git clone https://github.com/AUTOMATIC1111/stable-diffusion-webui`
* 3.双击`webui-user.bat`,执行webui安装，安装完成后即可通过浏览器访问。

## Linux系统
基于Debian（Debian-based）系统
通过输入以下命令，webui将会安装在当前目录里：
```
sudo apt install git python3.10-venv -y
git clone https://github.com/AUTOMATIC1111/stable-diffusion-webui && cd stable-diffusion-webui
python3.10 -m venv venv
```
通过以下命令安装和启动webui：
```
./webui.sh {your_arguments}
```
> 需要制定python版本为：3.10
```
cd stable-diffusion-webui

sudo pacman -S pyenv
pyenv install 3.10.6
pyenv local 3.10.6

python -m venv venv
```

> 按照官方python版本建议安装3.10版本，如果本地不是该版本建议先卸载后再安装该版本，否则可能会出现一些兼容性问题


## Web UI系统访问

启动stable-diffusion-webui后，在本地浏览器可以访问连接：http://127.0.0.1:7860/

### 设置中文界面
运行完毕后，复制链接 http://127.0.0.1:7860 进入浏览器。在浏览器的Extensions选项中，取消localization的勾选，点击Load from加载，选择zh_CN，点击右边的下载按钮Install。下载完成后，刷新页面，选择zh_CN文件，点击Apply settings保存，然后重启
具体参见：https://github.com/dtlnor/stable-diffusion-webui-localization-zh_CN


训练模型使用教程：https://www.bilibili.com/read/cv22823437/

### 下载模型

主流模型下载网站：    
* Hugging face是一个专注于构建、训练和部署先进开源机器学习模型的网站：https://huggingface.co/
* Civitai是一个专为Stable Diffusion AI艺术模型设计的网站，是非常好的AI模型库：https://civitai.com/
* 主流模型被删除可以去备用模型站下载：https://www.4b3.com

### 训练Lora模型
* 训练一个Lora模型参见：https://blog.csdn.net/weixin_45250844/article/details/130302817
* Lora训练模型，可以安装`kohya-ss`环境进行模型训练，训练好的模型可以直接拷贝到stable diffusion中进行使用，具体参见：
https://github.com/kohya-ss/sd-scripts

## 引用和参考文档    
* 官方网站：https://stablediffusionweb.com/
* Github官方项目wiki：https://github.com/AUTOMATIC1111/stable-diffusion-webui/wiki
* stable diffusion介绍教程：https://zhuanlan.zhihu.com/p/622238031
