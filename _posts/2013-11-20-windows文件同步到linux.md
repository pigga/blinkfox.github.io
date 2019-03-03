---
title: windows文件同步到linux
date: 2013-11-20 13:57:00 +0800 
layout: post
permalink: /blog/2013/11/20/windows文件同步到linux.html
categories:
  - Linux
tags:
  - rsync
  - 同步
---

同步windows服务器软件至linux服务器实现<br/>
##### 1、  一般linux上默认安装有rsync软件
###### 一．查看及安装
**查看是否安装rsync**<br/>
命令
```
# rpm –qa | grep rsync
```
假如出现对应的rsync版本，则说明对应的linux上已有rsync。<br/>
否则，需要手动下载安装。具体的方式有<br/>
软件安装过于简单，现在Linux各大发行版都提供这个软件包，当然您也可以自己编译安装，在目前的情况下，我看没太大的必要；<br/>
```
[root@linuxsir:beinan]$ sudo apt-get  install  rsync  注：在debian、ubuntu 等在线安装方法； 
[root@linuxsir:beinan]# slackpkg  install  rsync   注：Slackware软件包在线安装； 
[root@linuxsir:beinan]# yum install rsync    注：Fedora、Redhat等系统安装方法；
```
其它Linux发行版，请用相应的软件包管理方法来安装；如果是源码包，也就是用下面的办法；
```
[root@linuxsir:/home/beinan]# tar xvf  sync-xxxx.tar.gz 
或
sync-xxx.tar.bz2 
[root@linuxsir:/home/beinan]# cd  sync-xxx 
[root@linuxsir:/home/beinan/sync-xxx]# ./configure --prefix=/usr  ;make ;make install   注：在用源码包编译安装之前，您得安装gcc等编译工具才行；
```

###### 二、修改rsync的配置文件
  
可以看到rysnc服务是关闭的(disable = yes)，这里把它开启，把disable的值改为no
 
###### 三、rsync服务器的配置文件rsyncd.conf ；
我们可以参照rsyncd.conf.html。具体步骤如下；
```
[root@linuxsir:~]#mkdir /etc/rsyncd  注：在/etc目录下创建一个rsyncd的目录，我们用来存放rsyncd.conf 和rsyncd.secrets文件； 
[root@linuxsir:~]#touch /etc/rsyncd/rsyncd.conf  注：创建rsyncd.conf ，这是rsync服务器的配置文件； 
[root@linuxsir:~]#touch /etc/rsyncd/rsyncd.secrets  注：创建rsyncd.secrets ，这是用户密码文件； 
[root@linuxsir:~]#chmod 600 /etc/rsyncd/rsyncd.secrets  注：为了密码的安全性，我们把权限设为600；必须 
[root@linuxsir:~]#ls -lh /etc/rsyncd/rsyncd.secrets
 -rw------- 1 root root 14 2007-07-15 10:21 /etc/rsyncd/rsyncd.secrets 
[root@linuxsir:~]#touch /etc/rsyncd/rsyncd.motd
```
下一就是我们修改 rsyncd.conf 和rsyncd.secrets 和rsyncd.motd 文件的时候了；<br/>
rsyncd.conf 是rsync服务器主要配置文件，我们来个简单的示例；比如我们要备份服务器上的 /home 和/opt ，在/home中，我想把beinan和samba目录排除在外；<br/>
```
# Distributed under the terms of the GNU General Public License v2
# Minimal configuration file for rsync daemon # See rsync(1) and rsyncd.conf(5) man pages for help
# This line is required by the /etc/init.d/rsyncd script
uid = root  #这个用户是系统用户，当rsync客户端连接上服务器后，会映射成这个用户上传或下载
use chroot = no
max connections = 4 #最大允许并法链接数
strict modes = yes 
port = 873  #rsync服务对应的端口
 
[demo]                  ## 模块名字，自己命名必须指定且唯一
path = /home/pigga/test  #需同步的文件夹
comment = This is test
auth users = scihoo #rsync的用户名是客户端使用的，连接成功后会映射到上面的uid用户
uid = root  #这个用户是系统用户，当rsync客户端连接上服务器后，会映射成这个用户上传或下载
gid = root  #组名效果同上
secrets file = /etc/rsyncd/rsyncd.secrets #密码所在文件
read only = no #不是只读模式这样用户就有上传的权限了
list = yes      #用户具有list目录的权限，上传之后的目录可见，且列表展示
hosts allow = * #允许所有网段的地址连接至服务器，可以指定具体的ip以/分割
```
注：关于 auth users 是必须在服务器上存在的真实的系统用户，如果你想用多个用户，那就以,号隔开；比如 auth users = beinan , linuxsir
密码文件：/etc/rsyncd/rsyncd.secrets的内容格式；
用户名:密码   例：
Scihoo:scihoo
注：这里的密码值得注意，为了安全，你不能把系统用户的密码写在这里。比如你的系统用户 linuxsir 密码是 abcdefg ，为了安全，你可以让rsync 中的linuxsir 为 222222 。这和samba的用户认证的密码原理是差不多的；
rsyncd.motd 文件;
它是定义rysnc 服务器信息的，也就是用户登录信息。比如让用户知道这个服务器是谁提供的等；类似ftp服务器登录时，我们所看到的 linuxsir.org ftp ……。 当然这在全局定义变量时，并不是必须的，你可以用#号注掉，或删除；我在这里写了一个 rsyncd.motd的内容为：
+++++++++++++++++++++++++++ + linuxsir.org  rsync  2002-2007 + +++++++++++++++++++++++++++
###### 四．启动rsync 服务器及防火墙的设置；
4.1 启动rsync服务器；<br/>
启动rsync 服务器相当简单，–daemon 是让rsync 以服务器模式运行；
```
[root@linuxsir:~]#/usr/bin/rsync --daemon --config=/etc/rsyncd/rsyncd.conf
```
注：如果你找不到rsync 命令，你应该知道rsync 是安装在哪了。比如rsync 可执行命令可能安装在了 /usr/local/rsync/bin/rsync目录；也就是如下的命令；
```
[root@linuxsir:~]#/usr/local/rsync/bin/rsync --daemon --config=/etc/rsyncd/rsyncd.conf
```
4.2 rsync服务器和防火墙；<br/>
Linux 防火墙是用iptables，所以我们至少在服务器端要让你所定义的rsync 服务器端口通过，客户端上也应该让通过。
```
[root@linuxsir:~]#iptables -A INPUT -p tcp -m state --state NEW  -m tcp --dport 873 -j ACCEPT [root@linuxsir:~]#iptables -L  查看一下防火墙是不是打开了 873端口；
```
我在这里通过图形界面在防火墙上添加端口的方式。
查看是否开启了对应端口 也可以使用netstat –an 然后在另外的机器使用telnet判断是否可以通过防火墙。
另：查看系统进程。Ps –ef，查看是否开启rsync
##### 2、  window客户端
在windows机器上安装cwRsyncServer客户端，本地使用cwRsyncServer_4.0.6_Installer.zip。可在互联网下载；安装完毕后,我们写个批处理来实现下载和上传的功能上传:即将window指定路径的文件上传至linux<!--[if !supportLineBreakNewLine]--><!--[endif]-->
```
@ECHO OFF d: cd "Program Files\cwRsyncServer\bin"
rsync -vzrtopg –progress --delete /cygdrive/d/websoft/ scihoo@192.168.1.43::demo --password-file=/cygdrive/d/rsyncd.secrets
```
其中/cygdrive/e/表示的是windows的E盘
注：该部分d/websoft/加上/为不包含websoft的上传，去掉/为带有websoft文件夹的上传 d:/rsyncd.secrets的密码文件，里面只包含密码，不包含用户
下载:即从linux服务器指定路径下载到window客户端<!--[if !supportLineBreakNewLine]--><!--[endif]-->
```
@ECHO OFF d: cd "Program Files\cwRsyncServer\bin"
rsync -vrtopg --progress --delete scihoo@192.168.1.43::demo /cygdrive/d/websoft/
```
 
##### 3、密码每次都需要手动输入的解决方案

这算是个老问题了，每次在windows主机上通过cwrsync向服务端同步数据的时候都会遇到，这次总结记录下吧。错误代码为：
```
password file must be owned by root when running as root
```
在linux上设置rsync的时候，需要将passwordfile设置为600权限。所以在windows上我们也可以用其自带的chmod.exe 执行，其cwrsync客户端默认安装的位置是C:\Program Files\cwRsync\bin ，具体做法如下：
```
“C:\Program Files\cwRsync\bin”  600/cygdrive/c/etc/password.txt
```
执行完以后，如果还有错误提示，可以使用chown.exe命令将其文件的属主做下更改。具体操作如下：
服务端：
```
chmod.exe -c 600/cygdrive/c/etc/password.txt
chown.exe SvcCWRSYNC/cygdrive/c/etc/password.txt
```
SvcCWRSYNC为windows上的cwrsync-server安装时默认新建的一个用户。
客户端：
```
chmod.exe -c 600/cygdrive/c/etc/password.txt
chown.exe administrator /cygdrive/c/etc/password.txt
```
默认客户端上没有chown.exe这个命令，直接从cwrsync-server的安装路径里拷贝一个过来就可以用了。windows的默认用户一般都是administrator，如果你不是以administrator登录的，请将上面命令中的administrator改成你当前使用的用户名。<br/>
 
**问题**：<br/>
可能会遇到的问题：<br/>
_问题一_：<br/>
```
@ERROR: chroot failed rsync error: error starting client-server protocol (code 5) at main.c(1522) [receiver=3.0.3]
```
原因：<br/>
服务器端的目录不存在或无权限。<br/>
创建目录并修正权限可解决问题。<br/>
尝试通过运行  `setsebool -P rsync_disable_trans on`

_问题二_：<br/>
```
@ERROR: auth failed on module tee rsync error: error starting client-server protocol (code 5) at main.c(1522) [receiver=3.0.3]
```
原因：<br/>
服务器端该模块（tee）需要验证用户名密码，但客户端没有提供正确的用户名密码，认证失败<br/>
_问题三_：
```
@ERROR: Unknown module ‘tee_nonexists’ rsync error: error starting client-server protocol (code 5) at main.c(1522) [receiver=3.0.3]
```
原因：<br/>
服务器不存在指定模块。<br/>
提供正确的模块名或在服务器端修改成你要的模块以解决问题。
 
_问题四_：
```
rsync: link_stat "." (in *** ) failed: Permission denied (13)
```
*** 是指你同步的某一个文件夹模块的名字，一般在服务端进行同步时会碰到，这个问题是因为一些Linux的SELinux默认开启了Enforce模式，将其关闭即可 直接执行getenforce 0 , 问题即可解决  
 
_问题五_：
```
rsync: failed to connect to 218.107.243.2: No route to host (113)
rsync error: error in socket IO (code 10) at clientserver.c(104) [receiver=2.6.9]
```
原因：<br/>
对方没开机、防火墙阻挡、通过的网络上有防火墙阻挡，都有可能。关闭防火墙，其实就是把tcp udp的873端口打开。<br/>
 
_问题六_：
```
rsync error: error starting client-server protocol (code 5) at main.c(1524) [Receiver=3.0.7]
```
原因：<br/>
/etc/rsyncd.conf配置文件内容有错误。请正确核对配置文件。<br/>
 
_问题七_：
```
rsync: chown "" failed: Invalid argument (22)
```
原因：<br/>
权限无法复制。去掉同步权限的参数即可。(这种情况多见于Linux向Windows的时候)<br/>
 
_问题八_：
```
password file must not be other-accessible
continuing without password file
Password:
```
原因：<br/>
这是因为rsyncd.pwd rsyncd.secrets的权限不对，应该设置为600。如：chmod 600 rsyncd.pwd<br/>
 
编外：修改rsync服务端的端口
1、vi /etc/xinetd.d/rsync
```
service rsync
{
disable = no
socket_type     = stream
wait            = no
user            = root
server          = /usr/bin/rsync
server_args     = --daemon
port            = 9999 #加上这行
 
log_on_failure += USERID
}
```
2、vi /etc/services
```
rsync           9999/tcp                                # rsync 增加这行
rsync           9999/udp                                # rsync 增加这行
#rsync          873/tcp                         # rsync 注释这行
#rsync          873/udp                         # rsync 注释这行
```
3、测试
```
rsync -Rrav testfile $DIP::${MAIN} –port 9999
```
ubuntu下

vi /etc/default/rsync
```
RSYNC_OPTS='--port=80' #修改这行
```
启动rsync
 
```
/etc/init.d/rsync start
```