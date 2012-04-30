---
layout: page
title: Recent Posts
tagline:
---
{% include JB/setup %}

{% for post in site.posts limit: 5 %}
  <div class="post">
    <h1><a href="{{ post.url }}">{{ post.title }}</a></h1>
    {{ post.content }}
    <div class="post_footer">
      Posted {{ post.date | date: "%d %B %Y" }} | <a href="{{ post.url }}">Link to this post</a> | <a href="{{ post.url }}#disqus_thread">Comments?</a>
    </div>
  </div>  
  <hr />
{% endfor %}

{% include JB/comment-counts %}
