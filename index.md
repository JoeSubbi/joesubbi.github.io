---
layout: default
title: Joe Subbiani
description: Exploring Code and Alternate Dimensions
---

# My Posts
---

<p>
    <ul>
    {% for post in site.posts %}
        <li>
        <a href="{{ post.url }}">{{ post.title }}</a>
        </li>
    {% endfor %}
    </ul>
</p>

---

# Contacts
---
 - email: joe.subbiani@gmail.com