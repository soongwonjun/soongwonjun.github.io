---
layout: page
title: About me
permalink: /pages/about/
---

Soongwon Jun, A Backend Software Engineer.  
  
Focused NodeJS application, MSA architecture


<div class="about-contact-me">
  {% for social in site.author.links %}
    {{ social.title }} : <a href="{% if social.title == 'Email' %}mailto:{% endif %}{{ social.url }}">{{ social.url }}</a>  <br/>
  {% endfor %}
</div>