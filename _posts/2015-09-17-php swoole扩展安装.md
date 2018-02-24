---
layout: post
title: "php swoole扩展安装过程"
categories: php
description: 
keywords: php,swoole
---

## swoole安装部署

> 下载swoole-1.7.19.tgz
> http://pecl.php.net/package/swoole
> https://github.com/swoole/swoole-src/releases

```bash
cd /data/install
wget http://pecl.php.net/get/swoole-1.7.19.tgz
tar -zvxf swoole-1.7.19.tgz
cd swoole-1.7.19
```

> 编译安装

```bash
/usr/local/php-5.4.40/bin/phpize
./configure --with-php-config=/usr/local/php-5.4.40/bin/php-config
make
make install
```

> 加入php.ini

```bash
echo "[swoole]
extension = swoole.so" > /usr/local/php-5.4.40/etc/ext/swoole.ini
```
> 重启php-fpm

```bash
/etc/init.d/php-fpm restart
```

> 在phpinfo()中查看

