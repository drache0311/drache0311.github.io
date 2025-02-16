---
title: "Finance"
permalink: /finance/
classes: wide
category: finance
---

<ul>
    {% for post in site.posts %}
        {% if post.categories contains page.category %}
            <li>
                <a href="{{ post.url | relative_url }}">{{ post.title }}</a>
                <p>{{ post.date | date: "%B %d, %Y" }}</p>
                <span class="page__meta-readtime">
                    <i class="far {% if include.type == 'grid' and document.read_time and document.show_date %}fa-fw {% endif %}fa-clock" aria-hidden="true"></i>
                    {% if words < words_per_minute %}
                     {{ site.data.ui-text[site.locale].less_than | default: "less than" }} 1 {{ site.data.ui-text[site.locale].minute_read | default: "minute read" }}
                    {% elsif words == words_per_minute %}
                      1 {{ site.data.ui-text[site.locale].minute_read | default: "minute read" }}
                    {% else %}
                      {{ words | divided_by: words_per_minute }} {{ site.data.ui-text[site.locale].minute_read | default: "minute read" }}
                    {% endif %}
                </span>
                <p>{{ post.tags }}</p>
            </li>
        {% endif %}
    {% endfor %}
</ul>
