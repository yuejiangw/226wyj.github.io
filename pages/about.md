---
layout: page
title: About
description: Stay hungry, Stay foolish
keywords: Wu, Yuejiang
comments: true
menu: About
permalink: /about/
---

Hi, there.

My name is WU Yuejiang, a SWE at [Indeed, inc](https://www.indeed.com/).

I admire the art of elegant coding, and I'm trying my best to become a better software engineer.

Although I'm a slow walker, but I never walk backwards.

## Contact

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
