---
title: JAVAEE面试之Hibernate缓存
date: 2015-01-28 13:57:00 +0800 
layout: post
permalink: /blog/2015/01/28/JAVAEE面试之Hibernate缓存.html
categories:
  - JAVA
tags:
  - Hibernate
  - 缓存
---

Hibernate缓存分为两类：包括一级缓存(session级别)、二级缓存(sessionFactory级别)以及查询缓存。<br/>
##### 一、Session缓存（又称作事务缓存）：<br/>
缓存范围：缓存只能被当前Session对象访问。缓存的生命周期依赖于Session的生命周期，当Session被关闭后，缓存也就结束生命周期。这就是一级缓存。<br/>
Hibernate一些与一级缓存相关的操作（时间点）：<br/>
数据放入缓存：<br/>
###### 1. save()。当session对象调用save()方法保存一个对象后，该对象会被放入到session的缓存中。<br/>
###### 2. get()和load()。当session对象调用get()或load()方法从数据库取出一个对象后，该对象也会被放入到session的缓存中。<br/>
###### 3. 使用HQL和QBC等从数据库中查询数据。<br/>
 
 
##### 二、SessionFactory缓存（又称作应用缓存）：<br/>
 
缓存范围：缓存被应用范围内的所有session共享,不同的Session可以共享。这些session有可能是并发访问缓存，因此必须对缓存进行更新。缓存的生命周期依赖于应用的生命周期，应用结束时，缓存也就结束了生命周期，二级缓存存在于应用程序范围。<br/>

什么样的数据适合放到二级缓存中？<br/>
###### （1）经常被访问
###### （2）改动不大
###### （3）数量有限
###### （4）不是很重要的数据，允许出现偶尔并发的数据。
 
常用的二级缓存插件 
```
EHCache  org.hibernate.cache.EhCacheProvider 
OSCache  org.hibernate.cache.OSCacheProvider 
SwarmCahe  org.hibernate.cache.SwarmCacheProvider 
JBossCache  org.hibernate.cache.TreeCacheProvider 
```
demo:
###### 1、打开二级缓存：
为Hibernate配置二级缓存：
在主配置文件中hibernate.cfg.xml ：
```
<!-- 使用二级缓存 -->
<property name="cache.use_second_level_cache">true</property>
 <!--设置缓存的类型，设置缓存的提供商-->
<property    
  name="cache.provider_class">org.hibernate.cache.EhCacheProvider
</property>
```
###### 2、配置ehcache.xml
###### 3、使用二级缓存需要在实体类中加入注解（也可以在需要被缓存的对象中hbm文件中的<class>标签下添加一个<cache>子标签）：
需要ehcache-1.2.jar包:<br/>
还需要 commons_loging1.1.1.jar包<br/>
```
@Cache(usage = CacheConcurrencyStrategy.READ_WRITE)
```
二级缓存的使用策略一般有这几种：
```
read-only
nonstrict-read-write
read-write
transactional
```
注意：我们通常使用二级缓存都是将其配置成 read-only ，即我们应当在那些不需要进行修改的实体类上使用二级缓存，否则如果对缓存进行读写的话，性能会变差，这样设置缓存就失去了意义。<br/>

二级缓存缓存的仅仅是对象，如果查询出来的是对象的一些属性，则不会被加到缓存中去。<br/>