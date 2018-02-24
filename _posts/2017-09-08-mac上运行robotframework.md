---
layout: post
title: "mac上运行robotframework"
categories: mac
description: 
keywords: mac,robotframework
---

> 安装 robotframework

```sh
sudo pip install robotframework 
```

> 安装 wxPython

```sh
brew install wxpython
cd /Library/Python/2.7/site-packages
sudo ln -s /usr/local/Cellar/wxpython/3.0.2.0/lib/python2.7/site-packages/wx-3.0-osx_cocoa/wx wx
```



> 安装RIDE

```sh
https://pypi.python.org/pypi/robotframework-ride
cd robotframework-ride
python setup.py install
```



> 打开界面

```sh
/usr/local/bin/ride.py
```



```sh
用 ride.py 啟動 RIDE。

如果環境內沒有安裝符合要求的 wxPython，會發生下面的錯誤：

$ ride.py
wxPython not found.
You need to install wxPython 2.8.12.1 with unicode support to run RIDE.
wxPython 2.8.12.1 can be downloaded from http://sourceforge.net/projects/wxpython/files/wxPython/2.8.12.1/
如果遇到下面的錯誤，表示 wxPython 安裝的是 32-bit 的版本，連帶地 RIDE 也必須執行在 32-bit mode 下。(自行編譯 wxPython 就沒有這個問題)

$ ride.py
python should be executed in 32-bit mode with wxPython on OSX.
透過 VERSIONER_PYTHON_PREFER_32_BIT 環境變數可以讓 Python 執行在 32-bit mode：

$ export VERSIONER_PYTHON_PREFER_32_BIT=yes
$ ride.py
```

