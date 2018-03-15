---
layout: post
title: "Python写入Innodb引擎表失败"
categories: python
description: 
keywords: python
---

# Python写入Innodb引擎表失败

> 将表结构调整为Innodb引擎，结果python业务出现异常

## 异常现象

> python程序没有报错，并且有返回lastid，但数据库却没有这条记录(自增id有累加)

	...
	cursor = self.Mysql.cursor();
	cursor.execute(executesql, savedata)
	lastrowid = cursor.lastrowid
	cursor.close()
	self.Mysql.close()
	print lastrowid
	....

## 异常原因

> python和其他语言不一样，对Innodb引擎的表若没有进行commit操作数据则不会报错到db中

	...
	cursor = self.Mysql.cursor();
	cursor.execute(executesql, savedata)
	lastrowid = cursor.lastrowid
	cursor.close()
	print lastrowid
	self.Mysql.commit()
	self.Mysql.close()
	....

## 解决办法

1. 在sql语句后加一个COMMIT命令

	cursor.execute('insert into user (id, name) values (1, 'luchanghong);COMMIT;')

2. 给数据库连接conn预定一个commit

	conn = MySQLdb.connect(host = 'localhost', user = 'root', passwd = 'root', db = 'test')
	conn.autocommit(1)

3. 在释放数据库连接之前commit

	conn = MySQLdb.connect()
	cursor.execute('insert into ...')
	cursor.close()
	conn.commit()
	conn.close()