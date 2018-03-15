---
layout: post
title: "nginx代理出现cannot have URI 错误提示"
categories: nginx
description: 
keywords: nginx代理
---

# nginx代理出现cannot have URI 错误提示

> “proxy_pass” cannot have URI part in location given by regular expression, or inside named location, or inside “if” statement, or inside “limit_except” block …

## 错误原因

    location / {
        root /data/wwwroot/test_images;
        if ( !-e $request_filename) {
                proxy_pass http://172.17.163.80:8100/images/make?param=$request_uri;
        }
    }

> proxy_pass指令不能包含URI，相关情况分别是：正则表达式location；命名location；if；limit_except。
> 因为location中使用了正则表达式，所以在proxy_pass中不能包含URI part

## 解决方法

	location / {
	    root /data/wwwroot/test_images;
	    if ( !-e $request_filename) {
	            rewrite . /images/make?param=$request_uri break;
	            proxy_pass http://172.17.163.80:8100;
	    }
	}
	
	location / {
	    root  /home/wwwroot/www.hiapk.com/webroot/hiapk/game;
	    if ( !-e $request_filename){
	       rewrite . /MAKE-GAME-HIAPK$uri break;
	       proxy_pass http://qt.baidu.com;
	    }
	}

> 参考资料：http://www.udpwork.com/item/12552.html