---
title: oracle 连接 异常 Acquisition Attempt Failed!!! Clearing pending acquires
date: 2013-03-04 13:57:00 +0800 
layout: post
permalink: /blog/2013/03/04/oracle-连接-异常-Acquisition-Attempt-Failed!!!-Clearing-pending-acquires.html
categories:
  - 问题一箩筐
tags:
  - JAVA
  - Oracle
---

出错代码
```
com.mchange.v2.resourcepool.BasicResourcePool$AcquireTask@1429b9f -- Acquisition Attempt Failed!!! Clearing pending acquires. 
While trying to acquire a needed new resource, we failed to succeed more than the maximum number of allowed acquisition 
attempts (30). Last acquisition attempt exception: 
java.sql.SQLException: Io 异常: Connection refused(DESCRIPTION=(TMP=)(VSNNUM=186647040)(ERR=12519)(ERROR_STACK=(ERROR=

(CODE=12519)(EMFI=4))))
	at oracle.jdbc.dbaccess.DBError.throwSqlException(DBError.java:134)
	at oracle.jdbc.dbaccess.DBError.throwSqlException(DBError.java:179)
	at oracle.jdbc.dbaccess.DBError.throwSqlException(DBError.java:333)
	at oracle.jdbc.driver.OracleConnection.<init>(OracleConnection.java:404)
	at oracle.jdbc.driver.OracleDriver.getConnectionInstance(OracleDriver.java:468)
	at oracle.jdbc.driver.OracleDriver.connect(OracleDriver.java:314)
	at com.mchange.v2.c3p0.DriverManagerDataSource.getConnection(DriverManagerDataSource.java:135)
	at com.mchange.v2.c3p0.WrapperConnectionPoolDataSource.getPooledConnection(WrapperConnectionPoolDataSource.java:182)
	at com.mchange.v2.c3p0.WrapperConnectionPoolDataSource.getPooledConnection(WrapperConnectionPoolDataSource.java:171)
	at com.mchange.v2.c3p0.impl.C3P0PooledConnectionPool$1PooledConnectionResourcePoolManager.acquireResource

(C3P0PooledConnectionPool.java:137)
	at com.mchange.v2.resourcepool.BasicResourcePool.doAcquire(BasicResourcePool.java:1014)
	at com.mchange.v2.resourcepool.BasicResourcePool.access$800(BasicResourcePool.java:32)
	at com.mchange.v2.resourcepool.BasicResourcePool$AcquireTask.run(BasicResourcePool.java:1810)
	at com.mchange.v2.async.ThreadPoolAsynchronousRunner$PoolThread.run(ThreadPoolAsynchronousRunner.java:547)
```
首先分析，本地主机与数据库服务器的连接状态有无异常，无。再次分析你能否操作数据库服务器的数据库，使用pl/sql developer测试发现仍能正常连接。这个时候就要借助强大的网络了。百度、Google之后，发现引起这个现象的原因可能有数据库连接数的问题。具体操作
```
SQL> select count(*) from v$process; 

  COUNT(*) 
---------- 
        44 

SQL> select count(*) from v$session; 

  COUNT(*) 
---------- 
        39 
```
基本正常。但可能是由于该模块的操作比较复杂，所以考虑修改数据库连接数。代码：
```
alter system set processes=250 scope=spfile; 
```
运行之后，仍没有效果<br/>
后来找来高手分析了一下，修改数据库连接的jdbc.properties中的`cpool.maxPoolSize`为20.重启之后问题解决。其实修改数据库连接数应该也没有问题，我想应该是没有重启数据库服务的原因吧。写下该文章一方面留一备忘，另外希望大拿指正。