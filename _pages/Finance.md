---
title: "Finance"
layout: posts
permalink: /finance/
author_profile: true
classes: wide
category: finance
---

<ul>
    {% for post in site.posts %}
        {% if post.categories contains page.category %}
            <li>
                <a href="{{ post.url | relative_url }}">{{ post.title }}</a>
                <p>{{ post.date | date: "%B %d, %Y" }}</p>
            </li>
        {% endif %}
    {% endfor %}
</ul>
