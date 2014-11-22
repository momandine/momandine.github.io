---
layout: page
title: Eventually Consistent
tagline: by Amandine Lee
---
{% include JB/setup %}

One day I hope to eventually have access to all of the useful and interesting knowledge about computers, math, and their applications. Follow along as I try to get there. 

[About me](about.html)

-------------------------------------------------------

{% for post in site.posts %}
<p><a href="{{ post.url }}"><h3>{{ post.title }}</h3></a></p>
<p>{{ post.content | strip_html | truncatewords: 40 }}</p>
{% endfor %}
