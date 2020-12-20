---
layout: page
title: About
description: 坚持做好一件事
keywords: lizhibiao, 李志标
comments: true
menu: 关于
permalink: /about/
---

世界上最难的就是坚持做好一件事！

未来路很长，一起成长！

## 联系

{% for website in site.data.social %}
* {{ website.sitename }}：[@{{ website.name }}]({{ website.url }})
{% endfor %}
### 博主微信：lizb0907, 加好友入交流群 (备注:github博客)

![](/images/about/weixin.jpg)

## Skill Keywords

{% for category in site.data.skills %}
### {{ category.name }}
<div class="btn-inline">
{% for keyword in category.keywords %}
<button class="btn btn-outline" type="button">{{ keyword }}</button>
{% endfor %}
</div>
{% endfor %}
