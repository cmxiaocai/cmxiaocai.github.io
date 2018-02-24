---
layout: post
title: "android ADB 操作"
categories: android
description: 
keywords: android,android adb
---

> Android Debug Bridge，我们一般简称为adb，主要存放在sdk安装目录下的platform-tools文件夹中，它是一个非常强大的命令行工具，通过这个工具你能够与你的android设备进行交互



## adb命令格式

```
adb [-d|-e|-s <serialNumber>] <command>
```

```
-d: 让唯一连接到该PC端的真实安卓设备执行命令，如果发现USB中连接有多部设备，将会报错
-e: 让唯一连接到该PC端的模拟器执行命令，如果发现开启了多个模拟器，将会报错
-s：通过设备的序列号进行指定设备执行命令
```



## 常用操作

**查看当前连接设备**

> 　　I. device 设备已经成功连接到了adb-server
>
> 　  II. offline 设备并没有连接到adb或者没有响应
>
> 　 III. no device 并没有设备/模拟器连接

```
adb devices
```

```
$ ~ adb devices
List of devices attached
* daemon not running. starting it now at tcp:5037 *
* daemon started successfully *
emulator-5554	device
```



**查看当前adb版本**

```
adb　version
```

```
$ ~ adb version 
Android Debug Bridge version 1.0.39
Revision 3db08f2c6889-android
Installed as /usr/local/bin/adb
```



**给设备进行软件的安装**
> 两种格式
```shell
adb -s <serialNumber> install <path-to-apk>
adb -s <serialNumber> shell pm install [options] <PATH>
```
```Shell
$ ~ adb -s emulator-5554 install /work/work_wwwroot/tool/apk/External_LM_China_176TS.apk
/work/work_wwwroot/tool/apk/External_L...214.6 MB/s (301481420 bytes in 1.339s)
WARNING: linker: libdvm.so has text relocations. This is wasting memory and is a security risk. Please fix.
	pkg: /data/local/tmp/External_LM_China_176TS.apk
Success
```

**将数据从设备复制到PC中**

```
adb -s <serialNumber> pull <remote> <local>
```
```
adb -s emulator-5554 pull /sdcard/xxx.log /home/android_log/xxx.log
```

**将数据从PC复制到设备中**

```
adb -s <serialNumber>push <local> <remote>
```

**在手机中执行shell命令**

```
$ ~ adb shell
root@generic_x86:/ # pwd
/
root@generic_x86:/ # ls
acct
cache
config
```

**查看手机中的包**

```
pm list packages 
```

**屏幕录像**

> 例如，屏幕开始录像并且储存到/sdcard中，同时名字为demo.mp4
```
adb shell screenrecord /sdcard/demo.mp4
```

**关闭设备请求，开启设备**
```
adb shell stop
adb shell start
```

**设备关机以及重启**

```
adb reboot
adb shutdown
```


**参考资料**

> http://developer.android.com/tools/help/adb.html#IntentSpec