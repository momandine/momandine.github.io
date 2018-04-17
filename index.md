---
layout: page
title: Eventually Consistent
---
{% include JB/setup %}

A highly sporadic technical blog about the things I find interesting and want to gradually store in my brain. [About me](about.html)

-------------------------------------------------------

{% for post in site.posts %}
<p><a href="{{ post.url }}"><h2>{{ post.title }}</h2></a></p>
<p>{{ post.content }}</p>

-------------------------------------------------------
{% endfor %}

<script async defer src="https://www.recurse-scout.com/loader.js?t=2dfb623ea33038f1502fa3b187d9c7e5"></script>
