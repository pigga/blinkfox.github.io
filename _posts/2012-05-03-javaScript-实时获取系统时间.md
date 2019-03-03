---
title: javaScript 实时获取系统时间
date: 2012-05-03 13:57:00 +0800 
layout: post
permalink: /blog/2012-05-03-javaScript-实时获取系统时间.html
categories:
  - 问题一箩筐
tags:
  - JS
  - 系统时间
---

```
<script language="javascript" type="text/javascript">

	$(document).ready(function() {
		fillDate();  
	});

	function fillDate(){
		//日期 
		 var now = new Date(); //获取系统日期，即Sat Jul 29 08:24:48 UTC+0800 2006 
		 var yy=now.getFullYear();; //截取年，即2006 
		 var mm = now.getMonth()+1; //截取月，即07 
		 var dd = now.getDate(); //截取日，即29 
		 var cal = yy+"."+ mm +"."+dd;
		 //取时间 
		 var hh = now.getHours(); //截取小时，即8 
		 var mm = now.getMinutes(); //截取分钟，即34 
		var ss = now.getTime() % 60000; 
		//获取时间，因为系统中时间是以毫秒计算的， 所以秒要通过余60000得到。 
		ss = (ss - (ss % 1000)) / 1000; //然后，将得到的毫秒数再处理成秒 
		var clock = hh+':'; //将得到的各个部分连接成一个日期时间 
		if (mm < 10) clock += '0'; //字符串 
		clock += mm+':';  
		if (ss < 10) clock += '0';  
		clock += ss; 
		
		$(".date").html(cal + " " + clock);
		var timeID=setTimeout(fillDate,1000);
	}
</script>
```
感谢wucf2004，通过改进他的方法，才有了新的成果。 原文http://www.cnblogs.com/wucf2004/archive/2006/11/28/575323.html