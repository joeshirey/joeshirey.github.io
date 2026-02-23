---
layout: default
title: Joe Shirey | Writing
---

<section class="hero animate">
    <div class="container">
        <h1>Reflections & Explorations</h1>
        <p>A collection of thoughts on software engineering, leadership, and the craft of buildng great things.</p>
    </div>
</section>

<section class="container animate" style="animation-delay: 0.1s;">
    <div class="posts-list">
        {% for post in site.posts %}
            <div class="post-card" onclick="window.location.href='{{ post.url | relative_url }}'">
                <span class="date">{{ post.date | date: "%B %-d, %Y" }}</span>
                <h2>{{ post.title }}</h2>
                <div class="excerpt">
                    {{ post.excerpt | strip_html | truncatewords: 30 }}
                </div>
                <a href="{{ post.url | relative_url }}" class="read-more">Read Article <span>&rarr;</span></a>
            </div>
        {% endfor %}
    </div>
</section>
