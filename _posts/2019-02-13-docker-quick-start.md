---
layout:     post
title:      Docker简易教程
subtitle:   docker个人实践随记
date:       2019-02-13
author:     dushenzhi
header-img: img/post-sample-image.jpg
catalog: true
tags:
    - docker
    - 笔记
    - 容器
---

个人docker hub地址：

[https://hub.docker.com/r/dushenzhi](https://hub.docker.com/r/dushenzhi)



https://github.com/aerokube/selenoid-images.git



**启动docker**

``` bash
systemctl start docker
```

**设置开机自启动**

``` bash
sudo systemctl enable docker
```







拉取镜像

``` bash
docker pull selenoid/hub
```



查看本地已存在镜像

``` 
 ~ docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
selenoid/chrome     latest              d129fd30f416        4 weeks ago         861MB
selenoid/base       4.0                 0cfdba47572f        3 months ago        622MB
```



启动一个镜像容器

``` bash
docker run -d -i -t selenoid/chrome
```

可以用`-p`参数在启动容器的时候配置`宿主`机器与`容器`的端口映射，如将`容器`的`8080`端口映射到`宿主`机器`12345`端口如下：

``` bash
docker run -d -i -t -p 12345:8080 selenoid/chrome
```



查看本地已启动容器

``` bash
docker ps
```



进入容器内部命令行环境

``` bash
docker exec -it 1127f4c5bd8b /bin/bash
```

以root用户进入命名行环境请加`-u 0`或者`-u root`参数



将一个变更后的容器打包成为一个新的镜像

``` bash
docker commit 3b109952660c selenoid/chrome_jdk8_tomcat:v1.0
```







从docker镜像中拷贝出文件

``` bash
docker cp naughty_aryabhata:/home/selenium/screenshot.png ./screenshot.png
```



编写`Dockerfile`

``` bash
# base image
FROM selenoid/chrome

# MAINTAINER
MAINTAINER dushenzhi

# set user
USER root

# set timezone
ENV TZ=Asia/Shanghai \

# set locale
LANG=C.UTF-8

# running required command
RUN apt-get update \
# base image
    && apt-get install -y openjdk-8-jdk \
    && apt-get install -y vim \
    && apt-get install -y tomcat8 \
    && wget -O /tmp/apache-tomcat-8.5.37.tar.gz http://mirrors.shu.edu.cn/apache/tomcat/tomcat-8/v8.5.37/bin/apache-tomcat-8.5.37.tar.gz \
    && tar -xzf /tmp/apache-tomcat-8.5.37.tar.gz -C /root \
    && rm /tmp/apache-tomcat-8.5.37.tar.gz


```

通过Dockerfile打包新镜像

``` bash
docker build -t dushenzhi/chrome_jdk_tomcat:v1 .
```





