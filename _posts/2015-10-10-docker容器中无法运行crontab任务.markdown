---
layout: post
title:  "20151010 Docker容器中无法运行crontab任务"
date:   2015-10-10 17:10:00
categories: docker
author: xiaocai
github_repo_username: cmxiaocai
github_repo : cmxiaocai
comments: true
---

### 现象

> 采用官方的centos6镜像创建容器后，安装并启动了crontab服务，但发现crontab脚本没有触发。（暂不确定是镜像问题还是容器问题）
> 逐步排除最后发现是crontab安装了但却无法使用
> vim /etc/crontab

~~~bash
# Example of job definition:
# .---------------- minute (0 - 59)
# |  .------------- hour (0 - 23)
# |  |  .---------- day of month (1 - 31)
# |  |  |  .------- month (1 - 12) OR jan,feb,mar,apr ...
# |  |  |  |  .---- day of week (0 - 6) (Sunday=0 or 7) OR sun,mon,tue,wed,thu,fri,sat
# |  |  |  |  |
# *  *  *  *  * user-name command to be executed
*/1 * * * * root echo "xiaocai" >> /tmp/crontab_run.log 
~~~

<!-- more -->

### 解决方案

> vim /etc/pam.d/crond
> 将其中的required修改为sufficient

~~~bash
account    required   pam_access.so
account    include    password-auth
session    required   pam_loginuid.so
session    include    password-auth
auth       include    password-auth
~~~

*改为:*

~~~bash
account    sufficient   pam_access.so
account    include    password-auth
session    sufficient   pam_loginuid.so
session    include    password-auth
auth       include    password-auth
~~~

*restart后发现log写入成功*

~~~bash
service crond restart
~~~
