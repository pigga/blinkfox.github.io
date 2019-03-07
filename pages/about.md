---
layout: page
title: About
description: Yannis的个人博客，专注于平时遇到问题的整理及总结。可能是他人的源码，可能有自己的感悟及灵机一动。不求上天入地，只求兵来将挡水来土掩。
keywords: Yannis
comments: true
menu: 关于
permalink: /about/
---
Yannis的个人博客
记录平时遇到的问题，以及对部分技术的总结。<br/>
间或有个人的生活感悟，感想.
> 内容全为一言堂，若是有些地方有失公允。麻烦告知与我.
或者直接在文章后留言。

## 联系

{% for website in site.data.social %}
* {{ website.sitename }}：[@{{ website.name }}]({{ website.url }})
{% endfor %}

## Skill Keywords

{% for category in site.data.skills %}
### {{ category.name }}
<div class="btn-inline">
{% for keyword in category.keywords %}
<button class="btn btn-outline" type="button">{{ keyword }}</button>
{% endfor %}
</div>
{% endfor %}
