---
layout: post
title: "mac下applescript使用"
categories: mac
description: 
keywords: mac,applescript
---

> 使用AppleScript脚本可以通过命令方式操作mac电脑

## 运行方式

**1.shell脚本中调用**

> 比较常用的方式，可以配合shell脚本优雅的操作mac

```sh
#/bin/sh
osascript <<EOD
    display notification "提示消息..." with title "Launchd Test"
EOD
```

**2.使用脚本编辑器**

> 较复杂的script建议现在脚本编辑器中调试

```
运用程序 -> 实用工具 -> 脚本编辑器.app
```

```Sh
-- 右上角弹出提示消息
display notification "提示消息..." with title "Launchd Test"
```



## DEMO场景

> 发出声音

```sh
beep
say "检测到移动硬盘，开始备份数据..."
(*
  beep 次数
  say "messages" using "嗓音"
*)
```

> 打开一个应用

```sh
tell application "iTerm"
	-- activate
	set frontmost to true
end tell
(*
   activate 是激活当前app，set frontmost是置顶。
*)
```

> 连接服务器

```sh
-- tell application "Finder" to open location "smb://worker:xiaocai@10.99.196.55/data"
tell application "Finder"
	open location "smb://worker:xiaocai@10.99.196.55/data"
	activate
end tell
(*
   两种写法
*)
```

> 调节声音, 需要借助UI元素结构查看工具 `UI Browser`,`UIElementInspector`模拟GUI操作。

```sh
tell application "System Preferences"
	--activate  -- 调试的时候可去除测试
	set current pane to pane "com.apple.preference.sound"
end tell

tell application "System Events"
	tell application process "System Preferences"
		-- click checkbox "静音"  of window "声音" 
		set value of value indicator 1 of slider 1 of window "声音" to 0.1
	end tell
end tell

quit application "System Preferences"

return
```



## 开发技巧

**查找app接口**

> 不同的app会提供参数对外调用，可以通过字典查看

```sh
脚本编辑器.app -> file -> open Quickly... -> 选择app
```



**UI元素结构**

> app提供的参数无法满足时可以使用模拟GUI操作

- UI Browser (30天试用)
- UIElementInspector （官方免费）

```
大体的操作就是选择界面上的操作对象 -> 生成applescript动作
详见：http://www.jianshu.com/p/c6bec4103a5d
```

