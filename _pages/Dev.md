---
title: "Dev"
layout: default
permalink: /dev/
classes: wide
category: dev
---

<ul>
    {% for post in site.posts %}
        {% if post.categories contains page.category %}
            <li>
                <a href="{{ post.url | relative_url }}">{{ post.title }}</a>
                <div>
                    <p>{{ post.date | date: "%B %d, %Y" }}</p>
                    {% include page__meta.html %}
                </div>
            </li>
        {% endif %}
    {% endfor %}
</ul>
