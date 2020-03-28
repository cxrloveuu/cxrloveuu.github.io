---
layout: page
title: About
description: 打码改变世界
keywords: json Chan 节省钱
comments: true
menu: 关于
permalink: /about/
---

我是 jsonchan 节省钱的陈兴锐

一个往前端路上走的小学生。

学然后知不足,教然后知困。

## 联系

{% for website in site.data.social %}

- {{ website.sitename }}：[@{{ website.name }}]({{ website.url }})
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
