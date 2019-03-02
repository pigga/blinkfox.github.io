---
title: Ajax的get请求在IE浏览器中乱码问题的解决方案
date: 2016-12-30 13:57:00 +0800 
layout: post
permalink: /blog/2016/12/30/Ajax的get请求在IE浏览器中乱码问题的解决方案.html
categories:
  - 问题一箩筐
tags:
  - JS
---

在web请求中可能涉及到ajax的get请求，参数为中文的情况。在Chrome或者Firefox下，请求正常，但IE下返回结果不对。通过比对发现，在IE浏览器下的请求参数出现了乱码。
 
**解决方案**：
```
var url = CONTROLLER_URL + "/findResourceListByPage.json";
    return $http.get(encodeURI(url + "?" + params))
    .then(function(response) {
          return {
              'header' :[{
                   "key" : "title",
                   "name" : "资源标题"
                  }, {
                    "key" : "type",
                    "name" : "资源类型"
                  }
               ],
    'rows' : response.data.data.resultList,
    'pagination' : response.data.data.pagination,
    "sort-by" : "activeState",
    "sort-order" : "asc"
}
});
```
将get请求的url+param使用encodeURI方法进行转化。之后就可以正常进行参数传递了。