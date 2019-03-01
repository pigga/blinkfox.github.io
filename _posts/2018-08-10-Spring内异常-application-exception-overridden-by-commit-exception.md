---
title: Spring内异常-application-exception-overridden-by-commit-exception
date: 2018-08-10 13:57:00 +0800
layout: post
permalink: /blog/2018/08/10/Spring内异常-application-exception-overridden-by-commit-exception.html
categories:
  - 问题一箩筐
tags:
  - Spring
---

在执行某一操作时，意外发现自己定义的异常，无法被外面的Controller catch到。<br/>
追查发现在service内是可以正常打印异常信息，但外层Controller仅能拿到一个事务回滚的roolback异常。<br/>
仔细查看错误信息：
```
application exception overridden by commit exception
```
应用自定义异常被事务异常覆盖了。<br/>
 
那么如何正常的在外层捕获到自定义的异常呢？<br/>
**方法1.在对应的service上追加**
```
@Transactional(rollbackFor = DemoException.class)
```
**方法2.让自定义的异常继承RuntimeException（未校验，不过理论上是可以的）**
 
那么为什么会可以呢？<br/>
事情的原因是 only unchecked exceptions cause rollbacks in spring transactions.(Spring的默认回滚RuntimeException)
在Spring事务管理里仅会回滚非检查异常。<br/>
我们捕获了一些特殊的情况，处理完之后会自动转化为一个checked exception 。事务不会对该异常做回滚。后续的事务回滚会覆盖自定义的异常。