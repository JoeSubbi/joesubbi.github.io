---
layout: default
title: Index
---

# Index

stuff and more stuff

<p>
    <ul>
    {% for post in site.posts %}
        <li>
        <a href="{{ post.url }}">{{ post.title }}</a>
        </li>
    {% endfor %}
    </ul>
</p>

did my posts show?