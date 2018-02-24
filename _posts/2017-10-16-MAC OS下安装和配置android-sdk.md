---
layout: post
title: "macos下安装和配置android-sdk"
categories: mac,android
description: 
keywords: android-sdk
---

###安装

在MAC上安装android-sdk，标准的安装方法是使用`homebrew`，运行如下命令：

`brew update`

`brew install android-sdk`

如果遇到无法通过代理下载安装包的情况时，可以先手动下载[安装包](https://homebrew.bintray.com/bottles/android-sdk-24.4.1_1.el_capitan.bottle.tar.gz)， 然后，将安装包放到homebrew的缓存里，

`$ cp <path to download file> $(brew --cache android-sdk)`

再执行

`$ brew install android-sdk`

这样，就可以成功安装android-sdk了。

------

###配置

1、配置`.bash_profile`，如果使用的是`Oh-My-Zsh`，则配置`.zshrc`，在文件的结尾加上下面这句：

`export ANDROID_HOME="/usr/local/opt/android-sdk"`

然后`source ~/.zshrc`使其生效

2、安装辅助包

运行`android`命令，调出Android SDK Manager，安装下面这些选项：

- Tools
  - Android SDK Tools
  - Android SDK Platform-tools
  - Android SDK Build-tools
- Android 5.1.1 (API 22)
  - SDK Platform
  - Intel x86 Atom_64 System Image
- Extras
  - Android Suport Library
  - Intel x86 Emulator Accelerator(HAXM Installer)

3、安装HAXM(Hardware_Accelerated_Execution_Manager)

进入以下目录

`$ cd /usr/local/Cellar/android-sdk/24.4.1_1/extras/intel/Hardware_Accelerated_Execution_Manager/`

然后安装HAXM

`$ ./HAXM\ installation`

就这样，android模拟器就安装好了。

------

###配置模拟器

运行以下命令：

`$ android avd`

- 打开Android Virtual Device Manager
- 选择`Device Definitions`一项
- 选择需要的模拟器类型，点击`Create AVD`

创建完成AVD之后，回到`Android Virtual Devices`选项卡，选择创建好的模拟器，点击`Start`就可以启动模拟器了。