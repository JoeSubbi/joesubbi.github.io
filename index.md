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
        <a href="{{ post.url }}" style="color:purple">{{ post.title }}</a> - {{ post.day }}
        </li>
    {% endfor %}
    </ul>
</p>

---

You can get in contact with me at: joe.subbiani@gmail.com