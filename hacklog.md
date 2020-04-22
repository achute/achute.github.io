---
title: "dev/random > ./hacklog.py"
layout: page
permalink: /hacklog/
---

### Welcome To HackLog

Collection of various Dev / Security Musings.
Writeup on various CTFs / HTB

<ul class="posts">
    {% for post in site.posts %}
        <li>
            <a class="reserved" href="{{ post.url }}">{{ post.title }}</a> :: {{ post.date}} @ { {{ post.categories }} }
        </li>
    {% endfor %}
</ul>
