---
title: jquery 多个checkbox的联动选择
date: 2012-08-24 13:57:00 +0800 
layout: post
permalink: /blog/2012/08/24/jquery-多个checkbox的联动选择.html
categories:
  - 问题一箩筐
tags:
  - JQuery
  - checkbox
---

jquery实现。多级checkbox的联动问题解决办法。注意引入jquery啊。效果图如下：图片不知道怎么编辑进来，已经上传。可以看看样子。
```
<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Transitional//EN" "http://www.w3.org/TR/xhtml1/DTD/xhtml1-transitional.dtd">
<html xmlns="http://www.w3.org/1999/xhtml">
<head>
<meta http-equiv="Content-Type" content="text/html; charset=utf-8" />
<meta http-equiv="X-UA-Compatible" content="IE=EmulateIE7" />
<title></title>
<script src="jquery-1.6.2.js" type="text/javascript"></script>
<SCRIPT LANGUAGE="JavaScript" >
<!--
	function checkboxall(){ 

		if($("#oneCheck").attr("checked")=="checked"){	//全选
			$("#oneCheck").attr("checked",true);
			$("input[name='checkboxType']").each(function(){
			   $(this).attr("checked",true);
			 }); 
			 $("input[name='checkboxStyle']").each(function(){
			   $(this).attr("checked",true);
			 }); 
		}else{											//取消全选
			$("#oneCheck").attr("checked",false);
			 $("input[name='checkboxStyle']").each(function(){
			   $(this).attr("checked",false);
			 });
			$("input[name='checkboxType']").each(function(){
			   $(this).attr("checked",false);
			 }); 

		}

	}


	function checkBoxTwo(id){
		var tabId = "sec_"+id;
		var trId = "secid_"+id;
		$("#oneCheck").attr("checked",false);
		if($("#"+trId).attr("checked")=="checked"){
			$("#"+tabId+" :input[type='checkbox']").each(function(){
				$(this).attr("checked",true);
			});
		}else{
			$("#"+tabId+" :input[type='checkbox']").each(function(){
					$(this).attr("checked",false);
			});
		}
	}

	function check(trId){
		var arr = trId.split("_");
		var pid = "#secid_"+arr[1];
		//var tabId = "#sec_"+arr[1];
		$(pid).attr("checked",false);
		$("#oneCheck").attr("checked",false);
		//var len = $(tabId+" :input[type='checkbox']").length;
		//to do continue
	}
//-->
</SCRIPT>
</head>

<body>
	<table>
		<tr>
			<td colspan='2'><input type='checkbox' name='checkboxPar' id = 'oneCheck' value='' onclick='checkboxall();'>全选</td>
		</tr>

		<tr>
			<td ><input type='checkbox' name='checkboxStyle' id = 'secid_1' value=''  onclick="checkBoxTwo('1');"></td>
			<td >   类别1</td>
		</tr>
	</table>
	<table id = 'sec_1'>
		<tr>
			<td ><input type='checkbox' name='checkboxType' id = 'thirdId_1_1' value='' onclick="check('thirdId_1_1');"></td>
			<td >      类别1的1</td>
		</tr>
		<tr>
			<td ><input type='checkbox' name='checkboxType' id = 'thirdId_1_2' value='' onclick="check('thirdId_1_2');"></td>
			<td >      类别1的2</td>
		</tr>
	</table>

	<table>
		<tr>
			<td ><input type='checkbox' name='checkboxStyle' id = 'secid_2' value=''  onclick="checkBoxTwo('2');"></td>
			<td >   类别2</td>
		</tr>
	</table>
	<table id='sec_2'>
		<tr>
			<td ><input type='checkbox' name='checkboxType' id = 'thirdId_2_1' value='' onclick="check('thirdId_2_1');"></td>
			<td >      类别2的1</td>
		</tr>
		<tr>
			<td ><input type='checkbox' name='checkboxType' id = 'thirdId_2_2' value=' ' onclick="check('thirdId_2_2');"></td>
			<td >      类别2的2</td>
		</tr>
		<tr>
			<td ><input type='checkbox' name='checkboxType' id = 'thirdId_2_3' value=' ' onclick="check('thirdId_2_3');"></td>
			<td >      类别2的3</td>
		</tr>
	</table>
</body>
```
**解释说明**：页面中使用大量的table，除了带有id的两个外，别的只是为了格式需要。该部分仍不是太完善，没有反向的判断。比如所有三级全部选中，对应的二级也要选中。所有的二级选中，对应的一级也需选中。