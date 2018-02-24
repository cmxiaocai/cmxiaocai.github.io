---
layout: post
title: "使用pyinstaller打包python脚本"
categories: python
description: 
keywords: python
---

> 在做服务器部署或维护时需要将自己写的python脚本拷贝到目标服务器中执行，但生产环境服务器上python版本、依赖等都不一致。使用pyinstaller能够将python脚本打包成一个可执行文件在没有python环境的机器上运行（还能生成windows环境.exe可执行文件）。

## 安装

> 通过pip安装即可，也可以下载源码然后直接执行pyinstaller.py

```sh
pip install pyinstaller
```

## 使用

> 先创建一个测试文件 test.py

```Python
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

driver.close()
```



> 在test.py目录下执行pyinstaller

```sh
pyinstaller -F test.py
```

> 执行完成之后生成以下文件

```sh
xiaocaideMacBook-Pro:pyinstaller xiaocai$ ls
build		dist		test.py		test.spec
```

> 执行 dist/test 文件

```sh
xiaocaideMacBook-Pro:pyinstaller xiaocai$ dist/test
```



## 参数

> 详细参数: https://pyinstaller.readthedocs.io/en/stable/usage.html 

| -F, --onefile                    | 将结果打包成一个可执行文件                            |
| -------------------------------- | ---------------------------------------- |
| -D, --onedir                     | 将所有结果打包到一个文件夹中，该文件夹中包括一个可以执行文件和可执行文件执行时需要的依赖文件。此中方式是默认的打包方式 |
| -o DIR, --out=DIR                | 打包的结果文件所在的路径，不设置此项时，打包路径在pyinstaller安装包下面 |
| --additional-hooks-dir=HOOKSPATH | 有时候打包过程无错误，但是执行时报找不到某些依赖的包文件，就可以使用该参数设置，下面会看到该参数如何使用 |



```
pyinstaller还有一种打包方法，你使用命令行方式打包后目录下会生成一个.spec文件，你可以用文本框打开，修改一下，放到另外一个目录后，直接用pyinstaller <文件名>.spec 来生成转换exe文件， 我这有例子如下：（其实转换后最方便的我觉得还是py2exe,  如果程序转换有什么问题，会在目录下生成一个同名.log文件，问题都在文件里记录着。建议尽量不要打包成单文件，启动起来慢好多）

http://legendtkl.com/2015/11/06/pyinstaller/
```

