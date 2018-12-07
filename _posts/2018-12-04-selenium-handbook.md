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
        //options.setHeadless(true);//默认展示浏览器,该参数设置为true将不展示浏览器
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
