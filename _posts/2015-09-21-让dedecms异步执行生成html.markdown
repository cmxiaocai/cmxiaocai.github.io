---
layout: post
title:  "20150921 让dedecms异步执行生成html"
date:   2015-09-21 18:10:00
categories: dedecms
author: xiaocai
github_repo_username: cmxiaocai
github_repo : cmxiaocai
comments: true
---


>公司的一些旧项目采用开源的dedecms系统搭建，随着访问量与内容量堆积，各方面已经出现了瓶颈。
>dedecms采用生成静态html文件的方式发布内容，当有一篇新文章发布时需要重新生成上百页的栏目页面才能达到即时更新的效果。除了静态文件多台服务器共享存储网络瓶颈、还存在单台服务器的IO瓶颈。
>开源项目的替换需要有一个周期的过程，为此针对栏目列表页的生成进行了优化，采用后端异步多进程的方式处理，解决手工操作504现象。

*效果:*

![](http://www.xiaocai.name/uploads/201509/dedecms-makehtml1.jpg)

![](http://www.xiaocai.name/uploads/201509/dedecms-makehtml2.jpg)

压缩包下载:[http://www.xiaocai.name/uploads/201509/makelist_list_v2.zip](http://www.xiaocai.name/uploads/201509/makelist_list_v2.zip "makelist_list_v2.zip")

*后台守护进程:*

~~~php
swoole_timer_add(1100, function() {
    $process = new swoole_process(function($worker){
        $worker->exec('/usr/local/bin/php', array(__DIR__.'/service/makehtml_list.php'));
    }, true);
    $process->start();
    $process->wait();
});
~~~

<!-- more -->

### 解压makelist_list_v2.zip文件

>解压后得到以下文件,拷贝至dedecms目录

~~~
A       dede/makehtml_list_v2.php
A       dede/templets/makehtml_list_v2_end.htm
A       dede/templets/makehtml_list_v2_handle.htm
A       dede/templets/makehtml_list_v2_create.htm
A       service/makehtml_list.php
A       include/listmakehtml.class.php
~~~

### 原文件修改

> include/typelink.class.php
> 新增LogicGetIds方法

~~~php
    function LogicGetIds($id)
    {
        global $cfg_admin_channel;
        if(empty($cfg_admin_channel)) $cfg_admin_channel = 'all';
        $this->dsql->SetQuery("SELECT id,typename,typealias,ispart FROM #@__arctype WHERE reid='".$id."' AND ispart<>2 ORDER BY sortrank ASC");
        $this->dsql->Execute($id);
        while($row=$this->dsql->GetObject($id))
        {
            $this->LogicIds[$row->id] = $row->typename;
            $this->LogicGetIds($row->id);
        }
    }
~~~

> exit中断改为抛出异常
> include/arc.sglistview.class.php
> include/arc.listview.class.php

~~~php

echo "模板文件不存在，无法解析文档！";
exit();

改为

throw new Exception("模板文件不存在，无法解析文档！", 1123);

~~~

> 修改文章发布和编辑
> dede/article_add.php
> dede/article_edit.php

~~~php
    // - START
    // 此段加到返回成功信息之后
    require_once(DEDEINC."/listmakehtml.class.php");
    $datas = Xc_Memory::get('Task-TypeUpdate');
    if(!$datas){
        $datas=array();
    }
    $datas = array_merge($datas, array($synctypeid, $typeid, $typeid2));
    $datas = array_filter(array_unique($datas));
    Xc_Memory::set('Task-TypeUpdate', $datas, 2592000);
    // - END
~~~

> data/config.cache.inc.php
> 加入memcache配置

~~~php
$cfg_memcache_mc_defa = 'memcache://172.0.0.0:6379/def';
~~~