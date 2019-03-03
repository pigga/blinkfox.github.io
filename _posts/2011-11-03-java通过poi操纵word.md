---
title: java通过poi操纵word
date: 2011-11-03 13:57:00 +0800 
layout: post
permalink: /blog/2011/11/03/java通过poi操纵word.html
categories:
  - 问题一箩筐
tags:
  - JAVA
  - Word
  - POI
---

最近几天公司需要使用java处理报表，显示成word格式。有很多种处理方式，我采用了poi的处理。今天在做demo的时候遇到问题：word中的内容除了图片之外都可以读取到，然后我使用`range.replaceText("ak", "自己人");`替换word中的ak。打印代码显示成功替换，但是为什么我查看word,里面什么也没有啊。具体代码粘贴如下：
```
public class PoiDemo {
	public static void main(String[] args) {
//		writeDoc2("D:\\aaa.doc");
		try {
			HWPFDocument document=new HWPFDocument(new FileInputStream("D:\\aaa.doc"));
			Range range=document.getRange();
			range.replaceText("ak", "自己人");
			String str=range.text();
			System.out.println("--------->"+str);
			writeDoc("D:\\aaa.doc",str);
		} catch (FileNotFoundException e) {
			e.printStackTrace();
		} catch (IOException e) {
			e.printStackTrace();
		}
	}

	public static boolean writeDoc(String path,String string){
		boolean w=false;
		byte b[]=string.getBytes();
		FileOutputStream fs;
		try {
			fs = new FileOutputStream("D:\\aaa.doc");

			HWPFOutputStream  hos=new HWPFOutputStream();
			hos.writeTo(fs);
			hos.close();
			w=true;
		} catch (FileNotFoundException e) {
			e.printStackTrace();
		} catch (IOException e) {
			e.printStackTrace();
		}
		System.out.println("------->"+w);
		return w;
	}
}
```
当然你要上官网下载poi的jar。期待有高人知道问题的答案。