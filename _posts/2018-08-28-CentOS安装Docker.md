---
title: CentOS安装Docker
date: 2018-08-28 13:57:00 +0800
layout: post
permalink: /blog/2018/08/28/CentOS安装Docker.html
categories:
  - Docker
tags:
  - Docker
---
#### **CentOS上安装Docker**

##### 前置条件：
64-bit 系统<br/>
kernel 3.10+<br/>

###### 1.检查内核版本，返回的值大于3.10即可。
```
$ uname -r
```
![unamr-r](https://img-blog.csdn.net/20170629163943864?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzY4OTIzNDE=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)
###### 2.使用 sudo 或 root 权限的用户登入终端。
###### 3.确保yum是最新的
```
$ yum update
```
###### 4.添加 yum 仓库
 ```
tee /etc/yum.repos.d/docker.repo <<-'EOF'
[dockerrepo]
 
name=DockerRepository
 
baseurl=https://yum.dockerproject.org/repo/main/centos/$releasever/
 
enabled=1
 
gpgcheck=1
 
gpgkey=https://yum.dockerproject.org/gpg
 
EOF
```
![yum](https://img-blog.csdn.net/20170629164910967?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzY4OTIzNDE=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)
###### 5.安装 Docker
```
$ yum install -y docker-engine
```
安装成功后，使用docker version命令查看是否安装成功，安装成功后------如下图
![docker_version](https://img-blog.csdn.net/20170629165725357?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzY4OTIzNDE=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)
###### 6.启动docker
```
$systemctl start docker.service
```
###### 7.验证安装是否成功(有client和service两部分表示docker安装启动都成功了)
使用docker version命令查看
![service_version](https://img-blog.csdn.net/20170629170029388?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzY4OTIzNDE=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)
###### 8.设置开机自启动
```
$ sudo systemctl enable docker
```
到此为止docker就完全安装好了。

[]: "https://img-blog.csdn.net/20170629163943864?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzY4OTIzNDE=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center"