---
layout: default
title: Index
---

# Index

stuff and more stuff

<ul>
  {% for post in site.posts %}
    <li>
      <a href="{{ post.url }}">{{ post.title }}</a>
    </li>
  {% endfor %}
</ul>

did my posts show?