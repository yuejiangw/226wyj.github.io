---
layout: page
title: About
description: Stay hungery, Stay foolish
keywords: Wu, Yuejiang
comments: true
menu: About
permalink: /about/
---

Hi, there.

My name is WU Yuejiang, a MSCS student in NYU.

I admire the art of elegant coding, and I keep learning to achieve that.

Although I'm a slow walker, but I never walk backwards.

## Contact

<ul>
{% for website in site.data.social %}
<li>{{website.sitename }}：<a href="{{ website.url }}" target="_blank">@{{ website.name }}</a></li>
{% endfor %}
{% if site.url contains 'mazhuang.org' %}
<li>
微信公众号：<br />
<img style="height:192px;width:192px;border:1px solid lightgrey;" src="{{ assets_base_url }}/assets/images/qrcode.jpg" alt="闷骚的程序员" />
</li>
{% endif %}
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
