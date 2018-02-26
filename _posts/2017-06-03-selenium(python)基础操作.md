---
layout: post
title: "selenium(python)基础操作"
categories: selenium
description: 
keywords: selenium,python
---


## 使用pip安装selenium

> 使用pip安装

```sh
pip install selenium
```



## chrome驱动安装

> 下载chrome驱动(需要翻墙)，记得选择对应的操作系统下载浏览器驱动（uname -a查看系统）

```sh
http://docs.seleniumhq.org/download/
或者
https://sites.google.com/a/chromium.org/chromedriver/
```

> 这里选择chromedriver_mac64.zip

- chromedriver_linux32.zip
- chromedriver_linux64.zip
- chromedriver_mac64.zip
- chromedriver_win32.zip

> 下载并解压chromedriver_mac64.zip压缩包之后将`chromedriver`移动到`/usr/local/bin`目录下

```sh
mv chromedriver /usr/local/bin/.
```

> 若只安装了selenium没有安装chromedriver会出现以下错误`chromedriver executable needs to be in PATH`

```sh
xiaocaideMacBook-Pro:demo1 xiaocai$ python hello.py
Traceback (most recent call last):
  File "hello.py", line 3, in <module>
    dr = webdriver.Chrome()
  File "/Library/Python/2.7/site-packages/selenium/webdriver/chrome/webdriver.py", line 62, in __init__
    self.service.start()
  File "/Library/Python/2.7/site-packages/selenium/webdriver/common/service.py", line 81, in start
    os.path.basename(self.path), self.start_error_message)
selenium.common.exceptions.WebDriverException: Message: 'chromedriver' executable needs to be in PATH. Please see https://sites.google.com/a/chromium.org/chromedriver/home
```



## 简单的selenium脚本

> 创建一个hello.py文件，并执行它。

```sh
#!/user/bin/env python
# -*-coding:utf-8-*-

from selenium import webdriver
import time

driver = webdriver.Chrome()
driver.get("http://www.baidu.com")
driver.find_element_by_id('kw').send_keys('selenium')
driver.find_element_by_id('su').click()

# 需要等待页面元素加载完毕，否则会出现找不到元素
time.sleep(2)

# 嵌套查找
links = driver.find_element_by_id('content_left').find_elements_by_class_name('t')
for link in links:
    href  = link.find_element_by_tag_name('a').get_attribute('href')
    title = link.find_element_by_tag_name('a').text
    print title

print '----------------------'

# xpath查找
links = driver.find_elements_by_xpath('//div[@id=\'content_left\']/div/h3/a')
for link in links:
    print link.text
```

> 注意区分`find_element`和`find_elements`的使用差异，`find_elements`用于匹配多个元素，如`find_element_by_tag_name('a')` 和 `find_elements_by_tag_name('a')`



## 参考资料

```sh
selenium资料
https://www.gitbook.com/book/easonhan007/selenium-webdriver/details
selenium安装
http://blog.csdn.net/zxy987872674/article/details/53082896
```

