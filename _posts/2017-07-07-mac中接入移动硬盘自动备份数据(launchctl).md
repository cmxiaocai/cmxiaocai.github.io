---
layout: post
title: "mac中接入移动硬盘后自动备份数据"
categories: mac
description: 
keywords: mac,launchctl
---

> 因为工作关系需要频繁的将mac中的数据定期备份到移动硬盘中，于是想到利用launchctl监控`/Volumes`目录变动，来触发shell脚本实现插入usb移动硬盘后自动做增量备份。

## 1.创建plist配置文件

> 首先在`/Library/LaunchAgents`下创建自己的plist配置
>
> 我的移动硬盘名称是:external_disk ,因此每次插入usb的时候都会挂载`/Volumes/external_disk/`.所以用`WatchPaths`来监听这个目录
>
> 触发监控后执行`/work/_script/mac_backdisk/handle.sh`这个shell脚本

```sh
vim ~/Library/LaunchAgents/xiaocai.crontab.backdisk.plist
```

```Xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
  <key>Label</key>
  <string>xiaocai.crontab.backdisk</string>
  <key>ProgramArguments</key>
  <array>
    <string>/work/_script/mac_backdisk/handle.sh</string>
  </array>
  <key>WatchPaths</key>
  <array>
      <string>/Volumes/external_disk/</string>
  </array>
</dict>
</plist>
```



## 2.编辑shell脚本

> 需要对最后写入的时间进行判断，每间隔一定的时候在进行备份（因为监听的是`/Volumes/external_disk/`移动硬盘的根目录会导致频繁的触发WatchPaths）。
>
> 记得给handle.sh加上可执行权限

```sh
#/bin/sh
TIME=`date +"%Y-%m-%d %H:%M:%S"`
LANG=en_US.UTF-8
export LANG

# 判断是否接入移动硬盘
if [ ! -d "/Volumes/external_disk/" ]; then
    exit
fi
if [ ! -f "/tmp/.lastbackuptime" ]; then
    echo `date +%s` > /tmp/.lastbackuptime
fi

# 备份必须间隔3个小时
LAST_TIME=`cat /tmp/.lastbackuptime`
NOW_TIME=`date +%s`
if [ $[NOW_TIME-LAST_TIME] -lt 10800 ]; then
    exit
fi

# 写入最后备份时间
echo $TIME > /Volumes/external_disk/.lastbackuptime
echo $NOW_TIME > /tmp/.lastbackuptime

# 提示消息
osascript <<EOD
   display notification "检测到移动硬盘，开始备份数据..." with title "xiaocai的mac"
EOD

# 调用备份脚本 (这里是python写的备份工具,也可以使用cp、tar命令代替)
PATH=`dirname $0`
/usr/bin/python $PATH/bckdisk.py
```



## 3.将定时任务加入系统

```sh
# 装载
launchctl load ~/Library/LaunchAgents/xiaocai.crontab.backdisk.plist
# 移除
launchctl remove xiaocai.crontab.backdisk
```

