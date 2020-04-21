---
title: ./hacklog.py
layout: page
permalink: /
---
# dev/random > ./hacklog.py

Welcome To HackLog

<ul class="posts">
    {% for post in site.categories.posts %}
        <li>
            <a class="reserved" href="{{ post.url }}">{{ post.title }}</a>
        </li>
    {% endfor %}
</ul>
