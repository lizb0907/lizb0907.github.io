---
layout: page
title: About
description: 坚持做好一件事
keywords: lizhibiao, 李志标
comments: true
menu: 关于
permalink: /about/
---

大多数还轮不到拼天赋，因为我们努力的程度还远远没有达到需要去拼天赋的时候。

熟能生巧！

未来路很长，一起成长！

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
