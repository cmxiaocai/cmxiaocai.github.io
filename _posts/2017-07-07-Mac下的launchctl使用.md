---
layout: post
title: "mac下使用launchctl"
categories: mac
description: 
keywords: mac,launchd
---

> launchd是在Mac 上自带的一个执行定时任务的服务（相当于Linux 上的crontab），其通过plist 文件进行配置。launchd支持的属性有定时任务（StartCalendarInterval），启动运行程序(Startup Process)，监控文件变动（WatchPaths）等。



## 场景一 : 定时触发

**1.编辑测试脚本**

```sh
vim /work/_script/mac_test.sh
```

```sh
#/bin/sh
LOG=`date +"%Y-%m-%d %H:%M:%S"`
osascript <<EOD
    display notification "触发成功~当前时间 $LOG" with title "Launchd Test"
EOD
```



**2.Plist 配置**

```sh
cd ~/Library/LaunchAgents
vim xiaocai.crontab.test.plist
```

```sh
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
  <key>Label</key>
  <string>xiaocai.crontab.test.plist</string>
  <key>ProgramArguments</key>
  <array>
    <string>/work/_script/mac_test.sh</string>
  </array>
  <key>StartInterval</key>
  <integer>3</integer>
</dict>
</plist>
```



**3.将定时任务加入系统**

```sh
# 加载
launchctl load xiaocai.crontab.test.plist
# 移除
launchctl remove xiaocai.crontab.test.plist
# 马上启动(马上触发一次,用于测试)
launchctl start xiaocai.crontab.test.plist
```

*运行成功后每隔3秒将在右上角弹出提示*

![](http://qtimg.bdstatic.com/hiapk/news/201707/07/595f43841780a.jpg)



## 场景二 : 文件目录改变监听

> 当 WI-FI 变化时，`/Library/Preferences/SystemConfiguration/com.apple.airport.preferences.plist`文件将发生改变;
>
> 当接入外部存储设备时，`/Volumes/`目录发生改变；

**1.编辑测试脚本**

```sh
vim /work/_script/mac_test.sh
```

```sh
#/bin/sh
osascript <<EOD
    display notification "检测到目录发生变化" with title "Launchd Test"
EOD
```



**2.Plist 配置**

```sh
cd ~/Library/LaunchAgents
vim xiaocai.crontab.test.plist
```

```sh
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
  <key>Label</key>
  <string>xiaocai.crontab.test</string>
  <key>ProgramArguments</key>
  <array>
    <string>/work/_script/mac_test.sh</string>
  </array>
  <key>WatchPaths</key>
  <array>
      <string>/work/tmp_code/</string>
  </array>
</dict>
</plist>
```



**3.将定时任务加入系统**

```sh
# 加载
launchctl load xiaocai.crontab.test.plist
# 在目录下改变一个文件测试
echo "##" >> /work/tmp_code/x.log
# 查看运行状态
$ launchctl list | grep xiaocai
-	0	xiaocai.crontab.test
-	0	xiaocai.crontab.backdisk
-	0	xiaocai.crontab.wifi
```



## plist配置说明

> StartCalendarInterval、StartInterval、WatchPaths三者只选其一

```Xml
<?xml version="1.0" encoding="UTF-8"?>  
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">  
<plist version="1.0">  
  <dict>
    
    <key>Label</key>
    <!-- 名称，要全局唯一 -->
    <string>xyz.hanks.spider</string> 

    <!-- 要运行的程序， 如果省略这个选项，会把ProgramArguments的第一个
    元素作为要运行的程序 -->
    <key>Program</key>
    <string>/Users/hanks/run.sh</string>

    <!-- 命令， 第一个为命令，其它为参数-->
    <key>ProgramArguments</key>
    <array>
      <string>/Users/hanks/run.sh</string>
    </array>

    <!-- 运行时间 -->
    <key>StartCalendarInterval</key>
    <dict>

      <key>Minute</key>
      <integer>30</integer>

      <key>Hour</key>
      <integer>9</integer>

      <key>Day</key>
      <integer>1</integer>

      <key>Month</key>
      <integer>5</integer>

      <!-- 0和7都指星期天 -->
      <key>Weekday</key>
      <integer>0</integer>

    </dict>

    <!-- 运行间隔，单位为秒 -->
    <key>StartInterval</key>
    <integer>30</integer>

    <!-- 监控文件(目录)变化 -->
    <key>WatchPaths</key>
    <array>
        <string>/Library/Preferences/SystemConfiguration</string>
    </array>
    
    <!-- 标准输入文件 -->
    <key>StandardInPath</key>
    <string>/Users/hanks/run-in.log</string>

    <!-- 标准输出文件 -->
    <key>StandardOutPath</key>
    <string>/Users/hanks/run-out.log</string>

    <!-- 标准错误输出文件 -->
    <key>StandardErrorPath</key>
    <string>/Users/hanks/run-err.log</string>
  </dict>  
</plist>
```



