---
title: linux下rsync服务的搭建
date: 2014-03-28 13:57:00 +0800 
layout: post
permalink: /blog/2014/03/28/linux下rsync服务的搭建.html
categories:
  - Linux
tags:
  - rsync
---

linux确认rsync的安装及服务开放<br/>
#### 1、查看是否安装rsync
命令# rpm –qa | grep rsyn
出现图示信息，表明已安装了rsync.

假如没有出现对应的版本信息，则需要进行安装
 
#### 2、修改rsync的配置文件
##### 2.1 新建并修改rsync的配置文件
主要涉及的文件有 rsyncd.conf，rsyncd.secrets和rsyncd.motd <br/>
 
创建文件夹及文件
```
[root@******* ~]# mkdir /etc/rsyncd	注：在etc下创建rsyncd目录，保存rsyncd.conf和rsyncd.secrets;
[root@******* ~]# touch /etc/rsyncd/rsyncd.conf  注：rsync服务器的配置文件
[root@******* ~]# touch /etc/rsyncd/rsyncd.secrets 注：保存同步的用户密码文件
[root@******* ~]# chmod 600 /etc/rsyncd/rsyncd.secrets  注：为了密码安全性把权限设为600 必须
[root@******* ~]# ls -lh /etc/rsyncd/rsyncd.secrets 
-rw-------. 1 root root 0 Dec 10 17:46 /etc/rsyncd/rsyncd.secrets
[root@******* ~]# touch /etc/rsyncd/rsyncd.motd
```
修改文件内容<br/>
修改Rsyncd.conf的文件
```
# Minimal configuration file for rsync daemon
# See rsync(1) and rsyncd.conf(5) man pages for help
# This line is required by the /etc/init.d/rsyncd script
uid = root	#这个用户是系统用户 ，当rsync客户端连接上服务器后，会映射成这个用户上传或下载
gid = root
use chroot = no
max connections = 4	 #最大允许并法链接数
strict modes = yes
log file=/var/log/rsyncd.log
pid file=/var/run/rsyncd.pid
lock file=/var/run/rsyncd.lock
port = 873          #rsync服务对应的端口
[demo]              ## 模块名字，自己命名 必须指定且唯一
path = /usr/catd/mesContent  #需同步的文件夹
comment = This is test
auth users =rsyncChina  #rsync的用户名 是客户端使用的，连接成功后会映射到上面的uid户
uid = root #这个用户是系统用户 ，当rsync客户端连接上服务器后，会映射成这个用户上传或下载
gid = root #组名 效果同上
secrets file = /etc/rsyncd/rsyncd.secrets #密码所在文件
read only = no #不是只读模式 这样用户就有上传的权限了
list = yes #用户具有list目录的权限，上传之后的目录可见，且列表展示
hosts allow = 192.168.0.121  #该部分客户端ip
 rsyncd.secrets
```
编辑密码文件内容
```
[root@******* ~]# vi /etc/rsyncd/rsyncd.secrets 
rsyncChina:rsyncChina
```
 
注： 这里的密码值得注意，为了安全，你不能把系统用户的密码写在这里。比如你的系统用户 linuxsir 密码是 abcdefg ，为了安全，你可以让rsync 中的linuxsir 为 222222 ；
编辑修改rsyncd.motd
```
[root@******* ~]# vi /etc/rsyncd/rsyncd.motd 
  +++++++++++++++++++++++++++
  +  rsync  2009-2014 +
  +++++++++++++++++++++++++++
```
#### 3、启动rsync服务器及防火墙设置
##### 3.1启动rsync服务
```
[root@******* ~]# /usr/bin/rsync --daemon --config=/etc/rsyncd/rsyncd.conf
```
查看该服务是否启动<br/>
使用ps –ef 查看是否启动<br/>

正常启动。<br/>
或者查看端口是否开了873端口<br/>
``` 
[root@******* ~]# lsof -i:873
COMMAND   PID USER   FD   TYPE DEVICE SIZE/OFF NODE NAME
rsync   19499 root    3u  IPv4 166010      0t0  TCP *:rsync (LISTEN)
rsync   19499 root    5u  IPv6 166011      0t0  TCP *:rsync (LISTEN)
```
出现类似提示信息说明开启成功
##### 3.2 防火墙开启873端口
```
[root@******* ~]# iptables -A INPUT -p tcp -m state --state NEW  -m tcp --dport 873 -j ACCEPT
[root@******* ~]# iptables -L  注：查看一下防火墙是否打开了873端口
```
或者通过别的机器telnet查看对应机器是否开启873端口
#### 4、将rsync加入系统自启动
```
[root@******* /]# vi /etc/rc.d/rc.local 
#!/bin/sh
#
# This script will be executed *after* all the other init scripts.
# You can put your own initialization stuff in here if you don't
# want to do the full Sys V style init stuff.
 
touch /var/lock/subsys/local

wait
/usr/bin/rsync --daemon --config=/etc/rsyncd/rsyncd.conf &
```