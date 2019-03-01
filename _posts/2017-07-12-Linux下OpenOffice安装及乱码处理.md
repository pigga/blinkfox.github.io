---
title: BigDecimal小数保留两位，末尾为0的小数保留整数位
date: 2018-01-31 13:57:00 +0800 
layout: post
permalink: /blog/2018/01/31/BigDecimal小数保留两位，末尾为0的小数保留整数位.html
categories:
  - 问题一箩筐
tags:
  - JAVA
  - BigDecimal
---

##### 1. 从openoffice官网(http://www.openoffice.org/download/)下载linux相应的openoffice安装包(安装包有deb和rpm,linux BIT 对应32bit和64bit)。
##### 2. 安装包名 OOo_3.3.0_Linux_x86-64_install-rpm-wJRE_zh-CN.tar.gz
##### 3. 安装openoffice
将安装包放置linux中(例如放在openOffice文件夹下)
```      
 [root@InfoM188 /]# cd openOffice/
 [root@InfoM188 openOffice]#
 tar zxvf OOo_3.3.0_Linux_x86-64_install-rpm-wJRE_zh-CN.tar.gz    #解压安装包
```
 
解压后将其重命名为openOffice3.3
```         
[root@InfoM188 openOffice]# cd openoffice3.3/RPMS/
[root@InfoM188 RPMS]# rpm -ivh *.rpm        #安装所有rpm文件
[root@InfoM188 RPMS]# cd desktop-integration/
[root@InfoM188 desktop-integration]#
rpm -ivh openoffice.org3.3-redhat-menus-3.3-9556.noarch.rpm   #安装桌面应用程序
```
 
##### 4. 安装字体（主要解决在Word转Pdf乱码的问题）
```
[root@InfoM188 desktop-integration]# cd /usr/share/fonts/
新建simsun文件夹，将C:\WINDOWS\Fonts下的simsun.ttc拷贝到simsun目录
[root@InfoM188 simsun]# mkfontscale   #生成fonts.scale文件
[root@InfoM188 simsun]# mkfontdir     #生成fonts.dir文件
[root@InfoM188 simsun]# fc-cache      
```
 
##### 5. 启动openoffice
```
[root@InfoM188 /]# cd /opt/openoffice.org3/program/
[root@InfoM188 program]#
soffice -headless -accept="socket,host=127.0.0.1,port=8100;urp;" -nofirststartwizard &
```
##### 6. 卸载openoffice
```
[root@InfoM188 /]# rpm -e `rpm -qa |grep openoffice` `rpm -qa |grep ooobasis`
```
卸载可能存在openoffice卸载不完全,下次安装会提示有部分包已经存在,具体问题具体对待,卸载提示的包即可 rpm –e 包名