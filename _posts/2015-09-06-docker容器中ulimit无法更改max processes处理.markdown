---
layout: post
title:  "docker在容器中ulimit无法更改max processes"
date:   2015-09-06 10:00:00
categories: ulimit,docker
author: xiaocai
github_repo_username: cmxiaocai
github_repo : cmxiaocai
comments: true
---

## 在容器中ulimit无法更改max processes

**现象:**
> 1.从线上拉取nginx、php配置，用于在容器中运行与线上环境相同配置
> 2.启动nginx后'netstat -ltnp'没有发现80端口
> 3.从nginx的error.log日志发现以下异常

~~~
2015/09/06 02:03:40 [alert] 24#0: setrlimit(RLIMIT_NOFILE, 51200) failed (1: Operation not permitted)
2015/09/06 02:03:40 [alert] 25#0: setrlimit(RLIMIT_NOFILE, 51200) failed (1: Operation not permitted)
2015/09/06 02:03:40 [alert] 26#0: setrlimit(RLIMIT_NOFILE, 51200) failed (1: Operation not permitted)
~~~

> 4.发现系统ulimit与nginx设置不一致

~~~bash
[root@a47d6b8111ff logs]# ulimit -n
1024
[root@a47d6b8111ff logs]# cat /usr/local/nginx/conf/nginx.conf | grep rlimit
worker_rlimit_nofile 51200;
~~~

> 5.修改系统ulimit出错
~~~bash
[root@a47d6b8111ff logs]# ulimit -n 51200
bash: ulimit: open files: cannot modify limit: Operation not permitted
~~~

**解决方法**
> 1. 修改宿主机ulimit

~~~bash
ulimit -u 204800 -HSn 204800
~~~

> 2. docker run指定--ulimit参数

~~~bash
格式:
docker run --ulimit nofile=128:256 --ulimit nproc=32:64
参考:
docker run --name testimages_cms -i -t --ulimit nproc=51200:102400 --ulimit nofile=51200:102400 -p 8910:80 -v /data/www/dedecms_hiapk_com/:/data/wwwroot/  172.17.163.105:8500/online/cms.hiapk.com
~~~


**参考资料**
>http://docs.oracle.com/cd/E52668_01/E54669/html/ch27s17.html
>http://17173ops.com/2014/10/13/docker%E5%9F%BA%E7%A1%80%E4%B8%8E%E9%AB%98%E7%BA%A7.shtml#toc53


