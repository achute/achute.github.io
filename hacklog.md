---
title: "dev/random > ./hacklog.py"
layout: page
permalink: /hacklog/
---

### Welcome To HackLog

Collection of various Dev / Security Musings.
Writeup on various CTFs / HTB

<ol class="posts">
    {% for post in site.posts %}
        <li>
            <b>
            <a class="reserved" href="{{ post.url }}">{{ post.title }}</a>
            </b>
            <br/>
            {{ post.date | date: "%b %d, %Y"}} @
            {
            {% for cat in post.tags %}
              <a class="reserved" href="{{ site.baseurl }}category/#{{cat}}">{{cat}}</a>{% if forloop.last == false %},{% endif %}

            {% endfor %}
            }
        </li>
    {% endfor %}
</ol>
