---
title: BootStrap如何支持多模态框弹窗
date: 2018-07-06 13:57:00 +0800
layout: post
permalink: /blog/2018/07/06/BootStrap如何支持多模态框弹窗.html
categories:
  - 问题一箩筐
tags:
  - JS
  - 模态框
---

```
$(document).on('show.bs.modal', '.modal', function(event) {
    $(this).appendTo($('body'));
}).on('shown.bs.modal', '.modal.in', function(event) {
    setModalsAndBackdropsOrder();
}).on('hidden.bs.modal', '.modal', function(event) {
    setModalsAndBackdropsOrder();
});


function setModalsAndBackdropsOrder() {  
    var modalZIndex = 1040;
    $('.modal.in').each(function(index) {
        var $modal = $(this);
        modalZIndex++;
        $modal.css('zIndex', modalZIndex);
        $modal.next('.modal-backdrop.in').addClass('hidden').css('zIndex', modalZIndex - 1);
    });
    $('.modal.in:visible:last').focus().next('.modal-backdrop.in').removeClass('hidden');
}
```
 
该操作的可能后果：
对应的model将会被移动到body外创建，一旦页面使用单页面技术框架类似于AngularJs相关的话，将会造成路由再次跳回的时候，model重复加载。不太建议直接这样实现。
 
若多Model弹窗的话 还有一种就是修改对应的层级model对应的z-index。