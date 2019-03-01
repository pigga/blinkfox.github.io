---
title: 字符串前面或后面补零
date: 2017-08-23 13:57:00 +0800
layout: post
permalink: /blog/2017/08/23/List与String数组转换.html
categories:
  - 问题一箩筐
tags:
  - JAVA
---

List 转换为 String数组
```
List<String> list = new ArrayList<String>();    
list.add("a1");    
list.add("a2");    
String[] toBeStored = list.toArray(new String[list.size()]);  
```
String数组转List
```
String[] arr = new String[] {"1", "2"};  
List list = Arrays.asList(arr);
```
