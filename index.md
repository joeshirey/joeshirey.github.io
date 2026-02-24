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
                        <a href="{{ post.url | relative_url }}" class="read-more">Read More &rarr;</a>
                    {% endif %}
                </div>
                <div class="post-meta">
                    <span class="date">{{ post.date | date: "%B %-d, %Y" }}</span>
                    {% if post.tags.size > 0 %}
                        <div class="tags">
                            {% for tag in post.tags %}
                                <span class="tag">{{ tag }}</span>
                            {% endfor %}
                        </div>
                    {% endif %}
                </div>
            </div>
        {% endfor %}
    </div>
</section>
