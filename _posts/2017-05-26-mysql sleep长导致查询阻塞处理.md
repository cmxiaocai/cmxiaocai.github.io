---
layout: post
title: "mysql sleep过长导致查询阻塞"
categories: mysql
description: 
keywords: mysql
---

# mysql sleep过长导致查询阻塞

> 由于是跨机房访问数据库，造成链接时间过长，最终导致mysql查询阻塞。

## 故障现象

		mysql> show processlist;

		+----------+--------------+---------------------+-----------+-------------+---------+-----------------------------------------------------------------------+------------------+
		| Id       | User         | Host                | db        | Command     | Time    | State                                                                 | Info             |
		+----------+--------------+---------------------+-----------+-------------+---------+-----------------------------------------------------------------------+------------------+
		|        4 | mysync       | 10.199.100.26:30771 | NULL      | Binlog Dump | 1706714 | Master has sent all binlog to slave; waiting for binlog to be updated | NULL             |
		| 13843390 | dedewww      | 10.79.146.58:46876  | hiapkdede | Sleep       |       0 |                                                                       | NULL             |
		| 13983570 | dedewww      | 10.79.146.58:36786  | hiapkdede | Sleep       |       0 |                                                                       | NULL             |
		| 14068398 | dedewww      | 10.79.146.57:33184  | hiapkdede | Sleep       |      16 |                                                                       | NULL             |
		| 14071754 | dedewww      | 10.79.146.58:29120  | hiapkdede | Sleep       |       0 |                                                                       | NULL             |
		| 14073477 | dedewww      | 10.79.146.58:30321  | hiapkdede | Sleep       |     132 |                                                                       | NULL             |
		| 14073491 | dedewww      | 10.79.146.58:30325  | hiapkdede | Sleep       |     134 |                                                                       | NULL             |
		| 14073519 | dedewww      | 10.79.146.58:30342  | hiapkdede | Sleep       |     133 |                                                                       | NULL             |
		| 14073520 | dedewww      | 10.79.146.58:30344  | hiapkdede | Sleep       |     133 |                                                                       | NULL             |
		| 14073522 | dedewww      | 10.79.146.58:30355  | hiapkdede | Sleep       |     132 |                                                                       | NULL             |
		| 14073523 | dedewww      | 10.79.146.58:30356  | hiapkdede | Sleep       |     133 |                                                                       | NULL             |
		| 14073524 | dedewww      | 10.79.146.58:30358  | hiapkdede | Sleep       |     133 |                                                                       | NULL             |
		| 14073529 | dedewww      | 10.79.146.58:30366  | hiapkdede | Sleep       |     133 |                                                                       | NULL             |
		| 14073532 | dedewww      | 10.79.146.58:30369  | hiapkdede | Sleep       |     134 |                                                                       | NULL             |
		| 14073536 | dedewww      | 10.79.146.58:30375  | hiapkdede | Sleep       |     134 |                                                                       | NULL             |
		| 14073540 | dedewww      | 10.79.146.58:30377  | hiapkdede | Sleep       |     134 |                                                                       | NULL             |
		| 14073548 | dedewww      | 10.79.146.58:30390  | hiapkdede | Sleep       |     134 |                                                                       | NULL             |
		| 14073557 | dedewww      | 10.79.146.58:30393  | hiapkdede | Sleep       |     133 |                                                                       | NULL             |
		| 14073575 | dedewww      | 10.79.146.58:30428  | hiapkdede | Sleep       |     134 |                                                                       | NULL             |
		| 14073577 | dedewww      | 10.79.146.58:30430  | hiapkdede | Sleep       |     134 |                                                                       | NULL             |
		| 14073579 | dedewww      | 10.79.146.58:30431  | hiapkdede | Sleep       |     133 |                                                                       | NULL             |
		| 14073604 | dedewww      | 10.79.146.58:30464  | hiapkdede | Sleep       |     132 |                                                                       | NULL             |
		| 14073609 | dedewww      | 10.79.146.58:30468  | hiapkdede | Sleep       |     134 |                                                                       | NULL             |
		...

## 问题定位 & 解决

> 定位：发现wait_timeout配置为600s,此次改成10s重启服务后问题解决。


		mysql> show global variables like 'wait_timeout';
		+---------------+-------+
		| Variable_name | Value |
		+---------------+-------+
		| wait_timeout  | 600    |
		+---------------+-------+
		1 row in set (0.00 sec)

> 还有一个是 mysql> show variables like 'wait_timeout';（注意会话变量和全局变量的区别）

**修改:**

		mysql> set global wait_timeout = 10;


> 记得也要修改my.cnf里面的wait_timeout配置


		vim /home/mysql_data/my.cnf
		wait_timeout = 10


> 调整后正常情况复杂会慢慢将下来

