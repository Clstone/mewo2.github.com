---
layout: page
title: 
tagline:
---
{% include JB/setup %}

{% for post in site.posts limit: 5 %}
  <div class="post">
    <h1><a href="{{ post.url }}">{{ post.title }}</a></h1>
            <span>({{ post.date | date:"%Y-%m-%d" }})</span>
    {{ post.content }}
  </div>  
{% endfor %}
