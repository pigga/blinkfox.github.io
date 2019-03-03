---
title: 在Eclipse下使用ant,build项目报jpeg-does-not-exist
date: 2015-01-16 13:57:00 +0800 
layout: post
permalink: /blog/2015/01/16/在Eclipse下使用ant,build项目报jpeg-does-not-exist.html
categories:
  - 问题一箩筐
tags:
  - Eclipse
---

在Eclipse下使用Ant，build项目时，报package com.sun.image.codec.jpeg does not exist 错误。导致编译通不过。<br/>
环境：Eclipse Kepler 、Ant 1.6、JDK1.7.0<br/>
原因：在JDK1.7+时，Oracle不允许使用sun.*的jar。具体参见http://www.oracle.com/technetwork/java/faq-sun-packages-142232.html 。
为了防止被墙，加上图片

因为不能修改项目的原有代码，不能更换JDK版本，所以我使用的解决方案为：<br/>
在ANT中明确指定使用这个rt.jar。<br/>
另外还有一种，把rt.jar丢到工程的lib下，但我没成功。<br/>