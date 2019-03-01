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

```
public static void main(String[] args) {
	DecimalFormat df = new DecimalFormat("###.##");
    BigDecimal b1 = new BigDecimal("28.0109");
    BigDecimal b2 = new BigDecimal("28.00");
    System.out.println("小数格式化：" + df.format(b1));
    System.out.println("整数格式化：" + df.format(b2));
}
```