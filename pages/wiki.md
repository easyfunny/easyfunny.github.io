---
layout: page
title: Wiki
description: My fxcking good memories.
keywords: 维基, Wiki
comments: false
menu: Wiki
permalink: /wiki/
---

> Some Wikis are better than my good memories.

<ul class="listing">
{% for wiki in site.wiki %}
{% if wiki.title != "Wiki Template" %}
<li class="listing-item"><a href="{{ wiki.url }}">{{ wiki.title }}</a></li>
{% endif %}
{% endfor %}
</ul>
