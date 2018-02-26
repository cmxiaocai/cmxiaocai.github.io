---
layout: post
title: "macos搭建appium+android自动化测试环境"
categories: android
description: 
keywords: android,test,appium
---

> 网上的教程写了非常多安装步骤，其实只需要安装「Android Studio」和「appium」这两个dmg包即可节省掉一半的安装步骤。

## 环境安装

**下载Android Studio**

> 网络上大部分教程都是推荐只单独安装android sdk,单独安装sdk坑还是蛮多的..并且有些集成在android studio工具无法使用到。 

![](http://www.xiaocai.name/uploads/201711/15095480678841.jpg)

**安装appium**

> appium有两种安装方式，可以下载运行appium.dmg或者通过node命令安装
> appium.dmg会有两个版本（蓝色图标和紫色图标，紫色图标是Appium-desktop版本,蓝色图标是Appium-Server版本），Server版本已经不更新了，desktop版本将继承Server。（目前网上的教程大部分都是安装serverb版本）
> 另外重要的是AppiumDesktop可以定位元素信息，录制脚本！

```
Appium-Server下载地址：https://bitbucket.org/appium/appium.app/downloads/
Appium-desktop项目地址：https://github.com/appium/appium-desktop
```

![](http://www.xiaocai.name/uploads/201711/15095480855461.jpg)


**Appium-Python-Client**

> 依赖node 和 pip

```
pip install selenium
pip install Appium-Python-Client
```

> 写个python文件验证

```python
from selenium import webdriver
print 'ok'
```

## 运行安卓模拟器

**打开android studio**

> 随便创建一个空项目,能够进入到ide界面就行

![](http://www.xiaocai.name/uploads/201711/15095498789200.jpg)

**运行AVD Manager工具**

> Tools - Android - AVD Manager

![](http://www.xiaocai.name/uploads/201711/15095499761401.jpg)

**运行模拟器**

> 默认会有两个设备，也可以直接新创建设备

![](http://www.xiaocai.name/uploads/201711/15095501129264.jpg)

**模拟器**

> 看见这个界面就说明运行成功了

![](http://www.xiaocai.name/uploads/201711/15095502734609.jpg)


## 运行appium

**打开appium**

> 这里选择默认参数，然后点击start

![](http://www.xiaocai.name/uploads/201711/15095503762294.jpg)

**运行成功**

> 起了一个server，监听4723端口

![](http://www.xiaocai.name/uploads/201711/15095504239243.jpg)


## 跑测试用例

> 我们以默认的「计算器」作为demo

**查看app元素**

> 和做web自动化测试一样，得原找到目标对象才能进行操作。需要借助Hierarchy View工具。
> 打开方式  Tools - Android - Android Device Monitor

![](http://www.xiaocai.name/uploads/201711/15095506725773.jpg)

> 在模拟器中打开计算器,选中对应的包名。添加Hierarchy View窗口。

![](http://www.xiaocai.name/uploads/201711/15095508655842.jpg)

> 比如数字7，对应的id=digit7

![](http://www.xiaocai.name/uploads/201711/15095511220342.jpg)


**编写脚本文件**

```python
import os,time
from selenium import webdriver

desired_caps = {}
desired_caps['platformName'] = 'Android'
desired_caps['platformVersion'] = '4.4.2'
desired_caps['deviceName'] = 'Android Emulator'
desired_caps['appPackage'] = 'com.android.calculator2'
desired_caps['appActivity'] = '.Calculator'

driver = webdriver.Remote('http://localhost:4723/wd/hub', desired_caps)

for x in xrange(1,9):
    for j in xrange(1,9):
        x = str(x)
        j = str(j)
        print x+" x "+j+" = "
        driver.find_element_by_id("digit"+x).click()
        driver.find_element_by_id("mul").click()
        driver.find_element_by_id("digit"+j).click()
        driver.find_element_by_id("equal").click()
time.sleep(5)
driver.quit()
```
