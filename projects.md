---
layout: page
title: Projects
---

{% assign sorted = site.projects | reverse %}
{% for project in sorted %}
<h2 id="{{project.title}}" style="text-align:center;">{{ project.title }}</h2>
{% if project.video %}
<video class="center" autoplay loop mute>
<source src="/assets/{{ project.video }}" type="video/mp4" />
</video>
{% else %}
{% picture {{ project.image }} alt="{{ project.title }}" %}
{% endif %}
<p style="text-align:center;">{{ project.description }}</p>
<p style="text-align:center;"><a href="{{ project.url }}">Read more</a> or <a href="https://github.com/{{ project.repo }}">View on Github</a></p>
{% endfor %}

