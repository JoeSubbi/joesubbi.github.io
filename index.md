---
layout: default
title: Joe Subbiani
description: Exploring Code and Alternate Dimensions
---

# My Posts

<p>
    <ul>
    {% for post in site.posts %}
        <li>
        <a href="{{ post.url }}">{{ post.title }}</a> - {{ post.date | date_to_string }}
        </li>
    {% endfor %}
    </ul>
</p>

---
<!--
# My Pages

<p>
    <ul>
    {% for page in site.pages %}
        <li>
        <a href="{{ page.url }}" style="color:'#ffffff'">{{ page.title }}</a>
        </li>
    {% endfor %}
    </ul>
</p>
-->