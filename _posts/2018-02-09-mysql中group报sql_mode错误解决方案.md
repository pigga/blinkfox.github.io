---
title: mysql中group报sql_mode错误解决方案
date: 2018-02-09 13:57:00 +0800
layout: post
permalink: /blog/2018/02/09/mysql中group报sql_mode错误解决方案.html
categories:
  - 问题一箩筐
tags:
  - MySQL
---

Mysql中group时报sql_mode的解决办法
1.
```
vi /etc/my.cnf(Windows下是my.ini) 在[mysqld]下添加 
sql_mode=STRICT_TRANS_TABLES,NO_ZERO_IN_DATE,NO_ZERO_DATE,ERROR_FOR_DIVISION_BY_ZERO,NO_AUTO_CREATE_USER,NO_ENGINE_SUBSTITUTION
```
2. 查看sql_mode
```
select @@global.sql_mode;
```
去掉里面的ONLY_FULL_GROUP_BY
``` 
set @@global.sql_mode ='STRICT_TRANS_TABLES,NO_ZERO_IN_DATE,NO_ZERO_DATE,ERROR_FOR_DIVISION_BY_ZERO,NO_AUTO_CREATE_USER,NO_ENGINE_SUBSTITUTION';
```