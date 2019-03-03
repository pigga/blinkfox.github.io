---
title: index jsp jspInit
date: 2013-02-27 13:57:00 +0800 
layout: post
permalink: /blog/2013/02/27/index_jsp_jspInit.html
categories:
  - 问题一箩筐
tags:
  - JAVA
  - JSP
---

项目运行时报错：
```
java.lang.NullPointerException  at org.apache.jsp.index_jsp._jspInit(index_jsp.java:22)
```
出现该问题的原因大部分是由于`jsp-api.jar`和`servlet-api.jar`的问题。因为它们可能和tomcat中自带的jar包冲突。首先尝试删除这两个jar。再次编辑，应该就可以了。