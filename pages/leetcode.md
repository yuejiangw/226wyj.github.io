---
layout: page
title: LeetCode
description: 人越学越觉得自己无知
keywords: LeetCode
comments: false
menu: 维基
permalink: /leetcode/
---

> 记多少命令和快捷键会让脑袋爆炸呢？

<ul class="listing">
{% for blog in site.leetcode %}
{% if blog.title != "Wiki Template" and blog.topmost == true %}
<li class="listing-item"><a href="{{ blog.url }}{{ blog.url }}"><span class="top-most-flag">[置顶]</span>{{ blog.title }}</a></li>
{% endif %}
{% endfor %}
{% for blog in site.leetcode %}
{% if blog.title != "Wiki Template" and blog.topmost != true %}
<li class="listing-item"><a href="{{ blog.url }}{{ blog.url }}">{{ blog.title }}<span style="font-size:12px;color:red;font-style:italic;">{%if blog.layout == 'mindmap' %}  mindmap{% endif %}</span></a></li>
{% endif %}
{% endfor %}
</ul>
