---
layout: page
title: Posts
---

<section class="archive-post-list">
   {% for post in site.posts %}
       {% assign currentDate = post.date | date: "%Y" %}
       {% if currentDate != myDate %}
           {% unless forloop.first %}</ul>{% endunless %}
           <h2>{{ currentDate }}</h2>
           <ul>
           {% assign myDate = currentDate %}
       {% endif %}
       <li><span>{{ post.date | date: "%B %-d" }}</span> - <a href="{{ post.url }}">{{ post.title }}</a></li>
       {% if forloop.last %}</ul>{% endif %}
   {% endfor %}
</section>
