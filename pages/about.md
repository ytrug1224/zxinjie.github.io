---
layout: page
title: About
description: 未来不可欺
keywords: 欣杰, zouxinjie
comments: true
menu: 关于
permalink: /about/
---

我是邹欣杰

坚信熟能生巧，努力改变人生, 未来不可欺。

## 联系

qq： 165390188
email： zouxinjie126@gmail.com

## Skill Keywords

{% for category in site.data.skills %}
### {{ category.name }}
<div class="btn-inline">
{% for keyword in category.keywords %}
<button class="btn btn-outline" type="button">{{ keyword }}</button>
{% endfor %}
</div>
{% endfor %}
