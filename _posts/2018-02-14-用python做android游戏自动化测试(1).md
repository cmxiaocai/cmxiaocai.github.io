---
layout: post
title: "（一）用python做android游戏自动化测试"
categories: android
description: 
keywords: python,android,test,adb,opencv
---

> 游戏自动化测试痛点在于难以定位控件，这里使用图像识别替代控件定位的方式，来完成游戏的自动化测试。
> 当然也可以混合使用图像识别+控件定位的方式满足需求，在这里只分享用adb+opencv实现游戏自动化测试过程。

![](http://www.xiaocai.name/uploads/201702/gametest01/15185039335877.jpg)


## 一、测试原理

> 图像定位测试的核心思路其实是利用adb来操作设备，用opencv实现图像区域匹配，匹配成功后计算目标位置然后触发adb。
> 简单来说就是如下步骤：
> 截图 -> 图像识别 -> 计算位置 -> 点击位置

## 二、创建python运行环境

> virtualenv提供隔离的Python运行环境（如果不需要的话可以忽略此步骤）

**1.创建目录：**

```
mkdir venv_gametest
cd venv_gametest
```

**2.创建并运行独立python环境**

```
virtualenv --no-site-packages venv
source venv/bin/activate
```

## 三、安装依赖环境

**1. 安装android adb**

> 笔者是mac os系统，所以直接通过Homebrew安装。
> 也可以通过下载android SDK的方式手动安装这里不做说明，下载地址：developer.android.com

```shell
brew cask install android-platform-tools
```

**2. 测试adb**

> 连上手机的数据线或打开模拟器，查看能不能找到设备

```shell
(venv) $ demo5_game adb version
Android Debug Bridge version 1.0.32
(venv) $ demo5_game adb devices
List of devices attached
192.168.56.100:5555	device
```

> 更多的adb操作，可以查看 https://github.com/mzlogin/awesome-adb

**3. 安装opencv**

> opencv用来帮助我们完成图像的处理
> 只需要numpy、Matplotlib、opencv-python三个包
> 详见：[install opencv](http://www.opencv.org.cn/opencvdoc/2.3.2/html/doc/tutorials/introduction/table_of_content_introduction/table_of_content_introduction.html#table-of-content-introduction)

```
sudo pip install --upgrade setuptools
sudo pip install numpy Matplotlib
sudo pip install opencv-python
```

**4. 测试opencv**

> 选一张图片拷贝以下代码，用opencv打开测试是否安装成功

```
#!/usr/bin/env python
# coding=utf-8

import cv2 as cv

img = cv.imread("test.jpg")
cv.namedWindow("Image"
cv.imshow("Image",img)
cv.waitKey(0)
cv2.destroyAllWindows()
```

*运行结果:*
![](http://www.xiaocai.name/uploads/201702/gametest01/15185092443953.jpg)



## 四、adb截图操作

> 先看如何用adb命令进行截图操作，详见：[adb屏幕截图](https://github.com/mzlogin/awesome-adb#%E5%B1%8F%E5%B9%95%E6%88%AA%E5%9B%BE)

*先截图保存到设备里：*

```
adb shell screencap -p /sdcard/screencap.png
```

*然后将 png 文件导出到电脑:*

```
adb pull /sdcard/screencap.png
```

> 同样的使用python中`commands`模块也能实现相同的效果

```
commands.getstatusoutput('adb shell screencap -p /sdcard/screencap.png')
commands.getstatusoutput('adb pull /sdcard/screencap.png')
```

> 将adb的操作做个简单的封装，便于后面使用

*file: libs/adb.py*

```python
#!/usr/bin/env python
# coding=utf-8

"""ADB"""

__author__ = 'xiaocai'

import commands

class adbKit(object):

    def screenshots(self, serialNumber=None):
        self.command('shell screencap -p /sdcard/screencap.png', serialNumber)
        self.command('pull /sdcard/screencap.png', serialNumber)

    def command(self, cmd, serialNumber=None):
        cmdstr = 'adb '
        if serialNumber:
            cmdstr = cmdstr+'-s '+serialNumber
        (status, output) = commands.getstatusoutput(cmdstr+cmd)
        return [status, output]

```


## 五、图像匹配

> 刚刚完成了第一个截图环节，接下来我们开始尝试用opencv去匹配目标图像的位置
> 详见：[opencv文档](https://docs.opencv.org/3.0-last-rst/modules/refman.html)

**1. 选取一张截图**

> 我们利用上面封装好的`adbKit`对当前设备，进行一次截图操作。

*file: test.py*

```python
#!/usr/bin/env python
# coding=utf-8

"""demo"""

__author__ = 'xiaocai'

import sys, time, commands
from libs import adb

adbkit = adb.adbKit()
adbkit.screenshots()
```

> 运行完之后可以看到根目录下出现了`screencap.png`图片

```
.
├── libs
│   ├── __init__.py
│   ├── __init__.pyc
│   ├── adb.py
│   └── adb.pyc
├── screencap.png
└── test.py
```

> 接下来我们要实现第一个用例，如下图要识别到游戏中右上角的`X`按钮

![](http://www.xiaocai.name/uploads/201702/gametest01/15185116328339.jpg)

> 用截图工具将`x`截取，保持到`images/btn_close_full.png`(需要注意分辨率)
![](http://www.xiaocai.name/uploads/201702/gametest01/15185119007126.jpg)

**2. 载入图像**

> 读取截图和要匹配的`x`图像

```python
import cv2
target_img = cv2.imread("screencap.png")
find_img   = cv2.imread("images/btn_close_full.png")
```

**3. 图像匹配**

> 使用图像模板匹配`cv::matchTemplate()`，通过返回的cvMinMaxLoc计算结果
> 详见:[Template Matching](https://docs.opencv.org/3.2.0/d4/dc6/tutorial_py_template_matching.html)
> `matchTemplate`会将模板图像在源图像中进行滑动匹配（从左往右,从上往下）

*参数说明：*

> `image`是源图像，`templ`是模板图像，`method`是匹配算法

```
Python: cv2.matchTemplate(image, templ, method[, result]) → result
Parameters:	
image – Image where the search is running. It must be 8-bit or 32-bit floating-point.
templ – Searched template. It must be not greater than the source image and have the same data type.
result – Map of comparison results. It must be single-channel 32-bit floating-point. If image is  W \times H and templ is  w \times h , then result is (W-w+1) \times (H-h+1) .
method – Parameter specifying the comparison method (see below).
```

*代码:*

> 这里我们用`cv2.TM_CCOEFF_NORMED`
> 不同的算法对匹配结果会有所差异，具体可参考：[Python+OpenCV学习（7）---模板匹配](http://blog.csdn.net/firemicrocosm/article/details/48374979)

```
result = cv2.matchTemplate(target_img, find_img, cv2.TM_CCOEFF_NORMED)
print result
```

*结果:*

> 从打印的结果看到，matchTemplate会在模板块和输入图像之间寻找匹配最后返回一组匹配结果图像

```
[[ 0.07075607  0.08976483  0.10165194 ...  0.07626463  0.06611969
   0.04449887]
 [ 0.04982539  0.06268624  0.07642973 ...  0.06295352  0.05140355
   0.02827941]
 [ 0.01129027  0.02193917  0.03671272 ...  0.03954886  0.02578432
   0.00115839]
 ...
 [-0.08573136 -0.07838587 -0.06693702 ... -0.03226329 -0.02669666
  -0.03457748]
 [-0.08250767 -0.07598738 -0.06588773 ... -0.0265673  -0.02382514
  -0.02491279]
 [-0.07762627 -0.07059815 -0.06101505 ... -0.00854323 -0.0053194
  -0.00458878]]
```

> 每个匹配图像会有匹配度，比如找出匹配度>0.8的图像(如果需要一次匹配多个结果可以使用这个方法)

*示例:*

```python
#!/usr/bin/env python
# coding=utf-8

"""demo"""

__author__ = 'xiaocai'

import commands, cv2
import numpy as np

from libs import adb

adbkit = adb.adbKit()
adbkit.screenshots()

target_img = cv2.imread("screencap.png")
find_img   = cv2.imread("images/btn_close_full.png")

result = cv2.matchTemplate(target_img, find_img, cv2.TM_CCOEFF_NORMED)
loc    = np.where( result >= 0.5)
for pt in zip(*loc[::-1]):
    print pt
```

**4. minMaxLoc方法**

> 找到匹配图像之后我们需要使用`minMaxLoc`函数在给定的矩阵中寻找最大和最小值(包括它们的位置).

*代码:*

> 依次是:最小匹配度，最大匹配度，最小匹配位置，最大匹配位置

```
result = cv2.matchTemplate(target_img, find_img, cv2.TM_CCOEFF_NORMED)
min_val,max_val,min_loc,max_loc = cv2.minMaxLoc(result)
print min_val,max_val,min_loc,max_loc
```

*结果:*

> 这里的`(1019, 74)`就是我们模板在源图中的左上角坐标了

```
-0.381701976061 0.779404222965 (1181, 519) (1019, 74)
```

## 六、计算点击位置

> 上面我们通过`matchTemplate`和`minMaxLoc`方法已经获取到模板图像在源图中的坐标了，但这个坐标只是左上角的位置，实际点击应该是模板图像的中间位置

![](http://www.xiaocai.name/uploads/201702/gametest01/15185783539006.jpg)


> 首先我们需要先获取模板图像的尺寸

```
find_img   = cv2.imread("images/btn_close_full.png")
find_height, find_width, find_channel = find_img.shape[::]
```

> 根据max_loc结果计算出中间位置

```python
pointUpLeft   = max_loc
pointLowRight = (max_loc[0]+find_width, max_loc[1]+find_height)
pointCentre   = (max_loc[0]+(find_width/2), max_loc[1]+(find_height/2))
```

> 为了更直观些，我们把坐标在源图中点出来并显示出图片

```python
cv2.circle(target_img, pointUpLeft, 2, (0, 255, 255), -1)
cv2.circle(target_img, pointCentre, 2, (0, 255, 255), -1)
cv2.circle(target_img, pointLowRight, 2, (0, 255, 255), -1)
cv2.namedWindow("Image")
cv2.imshow("Image", target_img)
cv2.waitKey(0)
cv2.destroyAllWindows() 
```


*完整代码：*

```python
#!/usr/bin/env python
# coding=utf-8

"""demo"""

__author__ = 'xiaocai'

import commands, cv2
import numpy as np

from libs import adb

# 截图
adbkit = adb.adbKit()
adbkit.screenshots()

# 载入图像
target_img = cv2.imread("screencap.png")
find_img   = cv2.imread("images/btn_close_full.png")
find_height, find_width, find_channel = find_img.shape[::]

# 模板匹配
result = cv2.matchTemplate(target_img, find_img, cv2.TM_CCOEFF_NORMED)
min_val,max_val,min_loc,max_loc = cv2.minMaxLoc(result)

# 计算位置
pointUpLeft   = max_loc
pointLowRight = (max_loc[0]+find_width, max_loc[1]+find_height)
pointCentre   = (max_loc[0]+(find_width/2), max_loc[1]+(find_height/2))

# 画点
cv2.circle(target_img, pointUpLeft, 2, (255, 255, 255), -1)
cv2.circle(target_img, pointCentre, 2, (255, 255, 255), -1)
cv2.circle(target_img, pointLowRight, 2, (255, 255, 255), -1)

# 显示图片
cv2.namedWindow("Image")
cv2.imshow("Image", target_img)
cv2.waitKey(0)
cv2.destroyAllWindows() 
```


*结果：*

![](http://www.xiaocai.name/uploads/201702/gametest01/15185791428524.jpg)


## 七、触发点击

> 离成功只差最后一步了，先看看adb是如何模拟按键输入的
> 详见：[模拟按键/输入](https://github.com/mzlogin/awesome-adb#%E6%A8%A1%E6%8B%9F%E6%8C%89%E9%94%AE%E8%BE%93%E5%85%A5)

*命令:*

```
adb shell input tap <x> <y>
```

*python调用:*

```python
class adbKit(object):

    def screenshots(self, serialNumber=None):
        self.command('shell screencap -p /sdcard/screencap.png', serialNumber)
        self.command('pull /sdcard/screencap.png', serialNumber)

    def click(self, point, serialNumber=None):
        return self.command('shell input tap '+str(point[0])+' '+str(point[1]), serialNumber)

    def command(self, cmd, serialNumber=None):
        cmdstr = 'adb '
        if serialNumber:
            cmdstr = cmdstr+'-s '+serialNumber
        (status, output) = commands.getstatusoutput(cmdstr+cmd)
        return [status, output]
        
```

*测试:*

```python
adbkit = adb.adbKit()
adbkit.click(max_loc)
```

## 八、总结

> 回过头来看最后的代码，仅需30行即可完成（截图 -> 图像识别 -> 计算位置 -> 点击位置）流程，剩下我们可以思考下如何将整个测试环境串联起来。

**完整的编码：**

```
#!/usr/bin/env python
# coding=utf-8

"""demo"""

__author__ = 'xiaocai'

import commands, cv2
import numpy as np

from libs import adb

# 截图
adbkit = adb.adbKit()
adbkit.screenshots()

# 载入图像
target_img = cv2.imread("screencap.png")
find_img   = cv2.imread("images/btn_close_full.png")
find_height, find_width, find_channel = find_img.shape[::]

# 模板匹配
result = cv2.matchTemplate(target_img, find_img, cv2.TM_CCOEFF_NORMED)
min_val,max_val,min_loc,max_loc = cv2.minMaxLoc(result)

# 计算位置
pointUpLeft   = max_loc
pointLowRight = (max_loc[0]+find_width, max_loc[1]+find_height)
pointCentre   = (max_loc[0]+(find_width/2), max_loc[1]+(find_height/2))

# 点击
adbkit.click(pointCentre)
```


