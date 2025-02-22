---
title: "Finance"
layout: posts-grid
permalink: /finance/
classes: wide
category: finance
---

<section>
    <h1>Finance</h1>
    <section class="section__grid">
        {% for post in site.posts %}
            {% if post.categories contains page.category %}
                <article class="article__grid">
                    <a href="{{ post.url | relative_url }}">{{ post.title }}</a>
                    <p>
                        <time>{{ post.date | date: "%d.%m.%Y" }}</time>
                        {% include page__meta.html %}
                    </p>
                </article>
            {% endif %}
        {% endfor %}
    </section>
</section>
