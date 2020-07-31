---
layout: page
title: 简介
description: 打码改变世界
keywords: Bin Tu, 涂斌
comments: true
menu: 关于
permalink: /about/
---

## Who?

涂斌，江西南昌人。

开朗，幽默，乐天派，始终坚信「努利就是努利的回报」。

向死而生，如果不能把每一天当作最后一天来生活，就让我把每一天当作最后 20 年来看待！

## 联系

<ul>
{% for website in site.data.social %}
<li>{{website.sitename }}：<a href="{{ website.url }}" target="_blank">@{{ website.name }}</a></li>
{% endfor %}
</ul>

## Skill Keywords

{% for skill in site.data.skills %}
### {{ skill.name }}
<div class="btn-inline">
{% for keyword in skill.keywords %}
<button class="btn btn-outline" type="button">{{ keyword }}</button>
{% endfor %}
</div>
{% endfor %}
