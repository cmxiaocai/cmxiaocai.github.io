---
layout: post
title: "Jenkins构建Docker镜像-无法执行docker命令"
categories: linux,Jenkins,docker
description: 
keywords: docker,jenkins
---

## 问题

> 在Jenkins中执行docker build报错：Cannot connect to the Docker daemon. Is the docker daemon running on this host?


**脚本：**

```
#!/bin/sh
id
echo "start build..."
echo `docker ps`
```

**错误日志：**

```
Started by remote host 117.28.231.17
Building in workspace /var/lib/jenkins/workspace/test_dockerbuild02
[test_dockerbuild02] $ /bin/sh /tmp/jenkins7776024465780877752.sh
uid=994(jenkins) gid=991(jenkins) groups=991(jenkins)
start build...

Cannot connect to the Docker daemon. Is the docker daemon running on this host?

Finished: SUCCESS
```

## 解决方案

**修复：**

> 原因是jenkins用户没有加入docker的用户组

```
vim /etc/sudoers
------------
## Allow root to run any commands anywhere
root    ALL=(ALL)       ALL
jenkins ALL=(ALL)       ALL

usermod -G docker jenkins
(cat vim /etc/group 看docker是哪个用户组，或者直接加入root用户组)

service jenkins restart
(记得重启服务)
```


