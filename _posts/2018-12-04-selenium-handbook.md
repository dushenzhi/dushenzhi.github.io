---
layout:     post
title:      Selenium入门示例
subtitle:   Selenium笔记
date:       2018-12-04
author:     dushenzhi
header-img: img/post-sample-image.jpg
catalog: true
tags:
    - selenium
    - java
    - 爬虫
    - 浏览器
---

## 什么是Selenium

Selenium能够自动化控制浏览器.可用于Web应用自动化测试、web自动化管理任务和爬虫等，只要是需要浏览器自动化流程处理的都可以用Selenium来处理。

Selenium支持多种编程语言，包括：

* Java
* Python
* C#
* JavaScript
* PHP
* C#
* Objective-C
* 其他...

支持多种浏览器，包括：

* Chrome
* Firefox
* IE
* Opera
* Edge 
* Safari
* 支持远程代理驱动（remote-driver,支持集群）的方式

支持多操作系统：

* Microsoft Windows
* Apple OS X
* Linux

更多平台支持参见：[https://www.seleniumhq.org/about/platforms.jsp#browse](https://www.seleniumhq.org/about/platforms.jsp#browse)



官方网站：[https://www.seleniumhq.org/](https://www.seleniumhq.org/)

官方文档：[https://www.seleniumhq.org/docs/index.jsp](https://www.seleniumhq.org/docs/index.jsp)

## 示例

以下以java为示例

mavan依赖：

``` xml
<dependency>
    <groupId>org.seleniumhq.selenium</groupId>
    <artifactId>selenium-java</artifactId>
    <version>3.141.59</version>
</dependency>
```



安装浏览器的驱动，比如说需要驱动chrome浏览器，那么需要先安装chromeDriver二进制的驱动文件，Selenium项目通过绑定chromeDriver来驱动和操纵浏览器执行相关操作

Mac:  

``` bash
brew tap homebrew/cask && brew cask install chromedriver
```

Linux:

``` bash
sudo apt-get install chromium-chromedriver
```

Windows:

``` bash
choco install chromedriver
```

参见:[https://github.com/SeleniumHQ/selenium/wiki/ChromeDriver](https://github.com/SeleniumHQ/selenium/wiki/ChromeDriver)



Java示例代码：

``` java
import org.openqa.selenium.By;
import org.openqa.selenium.WebDriver;
import org.openqa.selenium.WebElement;
import org.openqa.selenium.chrome.ChromeDriver;
import org.openqa.selenium.chrome.ChromeOptions;
import org.openqa.selenium.support.ui.ExpectedCondition;
import org.openqa.selenium.support.ui.WebDriverWait;

public class SeleniumExample {
    public static void main(String[] args) {
        ChromeOptions options = new ChromeOptions();
        //options.addArguments("--headless");//默认展示浏览器,该参数设置为true将不展示浏览器
        //options.addArguments("--auto-open-devtools-for-tabs");//打开浏览器时自动开启开发者工具
        WebDriver driver = new ChromeDriver(options);
        driver.get("http://www.google.com");      //请求google首页
        WebElement element = driver.findElement(By.name("q"));//获取搜索框页面元素
        element.sendKeys("Cheese!");  //设置搜索关键字
        element.submit();//提交搜索请求
        System.out.println("Page title is: " + driver.getTitle());
        (new WebDriverWait(driver, 10)).until(new ExpectedCondition<Boolean>() { //页面响应回调处理
            public Boolean apply(WebDriver d) {
                return d.getTitle().toLowerCase().startsWith("cheese!");
            }
        });

        System.out.println("Page title is: " + driver.getTitle());

        driver.quit();
    }
}
```



## 服务器环境搭建

安装Chrome浏览器

``` bash
curl https://intoli.com/install-google-chrome.sh | bash
```



参见：

https://intoli.com/blog/installing-google-chrome-on-centos/



最新版ChromeDriver驱动下载，也可以参见上文用命令安装。

https://sites.google.com/a/chromium.org/chromedriver/downloads

http://chromedriver.storage.googleapis.com/index.html









## 集群模式

selenium集群模式需要用到`selenium-server.jar`包，服务功能实现由该jar包提供，当前最新版本下载地址为（有墙请自备梯子）：

``` bash
wget http://selenium-release.storage.googleapis.com/3.9/selenium-server-standalone-3.9.1.jar
```



启动节点(hub)服务：

``` bash
java -jar selenium-server-standalone-3.9.1.jar -role hub
```

hub服务启动后会打印出相应的信息，可以看到node节点默认注册地址为：

``` 
http://localhost:4444/grid/register
```

客户端(Java/python等程序)连接地址：

``` 
http://localhost:4444/wd/hub
```



启动节点(node)服务,指定hub服务为上述启动的服务：

``` bash
java -jar selenium-server-standalone-3.9.1.jar -role node  -hub http://localhost:4444/grid/register -browser browserName=chrome,maxInstances=5,platform=mac
```

`-browser`参数可以不加，默认允许最多并发启动`11`个浏览器实例，`5`个`firefox`、`5`个`chrome`和`1`个`IE`(`mac`系统为`1`个`safari`)（参见:[默认配置文件](https://github.com/SeleniumHQ/selenium/blob/master/java/server/src/org/openqa/grid/common/defaults/DefaultNodeWebDriver.json)）。此处测试mac环境，没有IE浏览器，强制用`-browser`参数指定当前启动的node服务的浏览器为`chrome`，最多创建`5`个实例，也可以指定多个浏览器或者不同版本的浏览器，参数还可以通过一个json配置文件来指定，更多细节可以参见：https://github.com/SeleniumHQ/selenium/wiki/Grid2

启动节点后也会打印出相应的信息，提示node节点已经注册到对应的hub服务，同时hub服务会打印出有node服务注册信息，可以尝试在多个机器上启动多个node服务，注意指定hub服务地址，扩展集群能力。

node服务默认服务端口为`5555`，在同一个机器上启动多个node服务注意要修改端口，为了避免冲突，通过如下-port参数指定新端口，如：

``` bash
java -jar selenium-server-standalone-3.9.1.jar -role node  -hub http://localhost:4444/grid/register -port 5556
```



客户端连接Java示例：

``` java
DesiredCapabilities capability = DesiredCapabilities.chrome();
WebDriver driver = new RemoteWebDriver(new URL("http://localhost:4444/wd/hub"), capability);
//driver do something...
```





