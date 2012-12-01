---
layout: page
title: Paul Mooring
---
{% include JB/setup %}

<div class="content">
{% for post in site.posts %}
    <div class="post">
      <h2><a href="{{ post.url }}">{{ post.title }}</a></h2>
      <div class="date"><h3>{{ post.date | date: "%B %d, %Y" }}</h3></div>
      <div class="extract">{{post.content | strip_html | truncatewords:80}}</div>
    </div>
    <hr />
  {% endfor %}
</div>