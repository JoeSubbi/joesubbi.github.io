---
layout: default
title: Joe Subbiani
description: Exploring Code and Alternate Dimensions
---

# My Posts

<p>
    <ul>
    {% for post in site.posts %}
        <h3 style="display: flex; justify-content: space-between;"><a href="{{ post.url }}">{{ post.title }}</a>
        <div style="font-size: 12pt; display: inline; text-align: right;">{{ post.date | date_to_string }}</div></h3>
        <p style="text-align: justify; font-size: 13pt;">{{post.description}}</p>
        <hr>
    {% endfor %}
    </ul>
</p>

# My Projects

<p>
    <ul>
    {% assign pages_list = site.pages | sort:"url" %}
    {% for node in pages_list %}
        {% if node.title != null %}
            {% if node.layout == "page" %}
                <h3><a href="{{ node.url | absolute_url }}">{{ node.title }}</a></h3>
                <p style="text-align: justify; font-size: 13pt;">{{ node.description }}</p>
                <hr>
            {% endif %}
        {% endif %}
    {% endfor %}
    </ul>
</p>

<!--
# Code Snippets
<p>
    <ul>
    {% assign pages_list = site.pages | sort:"url" %}
    {% for node in pages_list %}
        {% if node.title != null %}
            {% if node.layout == "code" %}
                <li>
                <a href="{{ node.url | absolute_url }}">{{ node.title }}</a>
                </li>
            {% endif %}
        {% endif %}
    {% endfor %}
    </ul>
</p>
-->