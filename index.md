---
layout: page
title: Eventually Consistent
---
{% include JB/setup %}

A highly sporadic technical blog about the things I find interesting, often systems and Python.
[About me](about.html)

-------------------------------------------------------

{% for post in site.posts %}
<p><a href="{{ post.url }}"><h2>{{ post.title }}</h2></a></p>
<p>{{ post.content }}</p>

-------------------------------------------------------
{% endfor %}
