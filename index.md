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

# My Pages

<p>
    <ul>
    {% assign pages_list = site.pages | sort:"url" %}
    {% for node in pages_list %}
        {% if node.title != null %}
            {% if node.layout == "page" %}
                <li>
                <a href="{{ node.url | absolute_url }}">{{ node.title }}</a>
                </li>
            {% endif %}
        {% endif %}
    {% endfor %}
    </ul>
</p>

---

# Code Snippets
<p>
    <ul>
        <li>
        <a href="{{ '/rotor-code/' | absolute_url }}">C# 4D Rotor Code</a>

        </li>
    </ul>
</p>