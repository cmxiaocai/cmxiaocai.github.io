---
layout: post
title: "(二) 用python做android游戏自动化测试"
categories: python
description: 
keywords: python,opencv,android
---

> 在上一篇《（一）用python做android游戏自动化测试》中，我们实现了最基础的(截图 -> 图像识别 -> 计算位置 -> 点击位置)步骤。在实际的测试过程中会一直重复的以上步骤，所以本次的目标是将这些重复的UI交互操作进行封装。


## 创建UI交互类

> 首先在`adb.py`同级目录创建`uiDriver.py`文件,并申明`uiDriver`类

*目录结构参考:*

```
.
├── images
│   └── btn_close_full.png
├── libs
│   ├── __init__.py
│   ├── adb.py
│   └── uiDriver.py
└── run.py
```

*file: libs/uiDriver.py*

```python
#!/usr/bin/env python
# coding=utf-8

"""Android UI交互操作"""

__author__ = 'xiaocai'

import sys, time, cv2
from libs import adb

class uiDriver(object):

    def __init__(self):
        self.adb = adb.adbKit()
    
```

## find函数封装

> 接着我们要将上一篇的(截图 -> 图像识别 -> 计算位置)操作步骤封装到find方法中。另外新增一个precision参数用于设定匹配图像的相似度，这样可以在某些特殊场景下通过降低相似度达到匹配效果。

```python
...
class uiDriver(object):
    ...
    def find(self, img, precision=0.9):
        # 截图
        self.adb.screenshots()

        # 载入图像
        targetImg = cv2.imread("screencap.png")
        findImg   = cv2.imread("images/"+img)
        findHeight, findWidth, findChannel = findImg.shape[::]

        # 模板匹配
        result = cv2.matchTemplate(targetImg, findImg, cv2.TM_CCOEFF_NORMED)
        minVal,maxVal,minLoc,maxLoc = cv2.minMaxLoc(result)
        print 'find: '+img, maxVal, maxLoc

        # 计算位置
        pointUpLeft   = maxLoc
        pointLowRight = (maxLoc[0]+findWidth, maxLoc[1]+findHeight)
        pointCentre   = (maxLoc[0]+(findWidth/2), maxLoc[1]+(findHeight/2))

        # 匹配度要求
        if maxVal <= precision:
            print 'Can\'t find '+img, '(max:'+str(maxVal), 'expect:'+str(precision)+')'
            return False
                

        return pointCentre
```

> 封装完之后run.py主逻辑上的代码更加精简了

```python
#!/usr/bin/env python
# coding=utf-8
"""demo"""

__author__ = 'xiaocai'

import commands, cv2
from libs import adb,uiDriver

uiKit  = uiDriver.uiDriver()
adbKit = adb.adbKit()

pos = uiKit.find('btn_close_full.png')
if pos:
    adbkit.click(pos)

```

> 还可以将find和click操作合并在一起，避免主逻辑上有太多的if判断

```python
...
class uiDriver(object):
    ...
    def findClick(self, img, precision=0.9):
        point = self.find(img, precision)
        if not point:
            return False
        self.adb.click(point)
        return True
```

> 最后我们通过一个简单的case来验证一下程序是否能够走通，“在《王国纪元》中分别打开关闭「联盟」和「设置」面板”，如下图：

![](http://cmxiaocai.github.io/uploads/201803/15211699400595.jpg)

> 通过截图工具将这两个按钮保存下来(注意分辨率)，从截图中可以看出这两个按钮的上半部分会有一些干扰所以这里只截取了icon的下半部份作为匹配图像。

![](http://cmxiaocai.github.io/uploads/201803/15211691847029.jpg)

> 通过封装好的代码实现

```python
uiKit.findClick('btn_seting.png')
uiKit.findClick('btn_close_full.png')
uiKit.findClick('btn_task.png')
uiKit.findClick('btn_close_full.png')
```

## wait函数封装

> 通过上面的例子我们已经可以通过`findClick()`之间的调用，连贯的完成一系列的操作。但游戏场景中存在数据加载、延迟、性能等等干扰因素，会使find方法错过最佳图像匹配时机。因此我们需要在`find()`与`find()`之间加入一个预期的等待时间。

> 为uiDriver类新增wait函数，通过设置检测间隔和超时时间实现等待。大致如下：

```python
...
class uiDriver(object):
    ...
    def wait(self, img, interval=0.5, timeout=2, precision=0.9):
        startTime = time.time()
        while True:
            point = self.find(img, precision)
            if point:
                return point
            diff = time.time() - startTime
            if diff >= timeout:
                return False
            time.sleep(interval)
```

> 继续以《王国纪元》为例子验证一个简单的CASE，游戏载入成功之后检测并关闭弹窗，场景如下图：

![](http://cmxiaocai.github.io/uploads/201803/15216153931885.jpg)

**示例:**
    
```python

point = uiKit.wait(
    img='btn_close_full.png',
    interval=2,
    timeout=10
)
if point:
    uiKit.click(point)
    
```

## 总结

> 本章我们通过简单的封装uiDriver类可以使主干代码更加精简可读比如以下代码，不过用心的同学会发现这段代码在`异常`的情况下会很难定位到`问题`，在下一章会向大家讲解如何生成测试报表，感兴趣的同学可以继续关注博客。

```python
#!/usr/bin/env python
# coding=utf-8

"""demo"""

__author__ = 'xiaocai'

import commands, cv2
from libs import adb,uiDriver

uiKit  = uiDriver.uiDriver()
adbKit = adb.adbKit()

'''
CASE01: 游戏载入后关闭弹窗
'''
point = uiKit.wait(
    img='btn_close_full.png',
    interval=2,
    timeout=10
)
if point:
    uiKit.click(point)

'''
CASE02: 打开/关闭面板
'''
uiKit.findClick('btn_seting.png')
uiKit.findClick('btn_close_full.png')
uiKit.findClick('btn_task.png')
uiKit.findClick('btn_close_full.png')

'''
CASE N: 以下省略N个CASE
'''
...

print 'successful.'
```