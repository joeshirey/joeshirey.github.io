---
layout: default
title: Joe Shirey | Writing
---



<section class="container">
    <div class="posts-list">
        {% for post in site.posts %}
            <div class="post-entry">
                <div class="post-info">
                    <h2><a href="{{ post.url | relative_url }}">{{ post.title }}</a></h2>
                    {% if post.excerpt %}
                        <p class="excerpt">{{ post.excerpt | strip_html }}</p>
                    {% endif %}
                </div>
                <span class="date">{{ post.date | date: "%B %-d, %Y" }}</span>
            </div>
        {% endfor %}
    </div>
</section>
