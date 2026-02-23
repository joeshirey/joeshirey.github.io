---
layout: default
title: Joe Shirey | Writing
---

<section class="hero">
    <div class="container">
        <h1>Reflections & Explorations</h1>
        <p>Thoughts on software engineering, leadership, and craft.</p>
    </div>
</section>

<section class="container">
    <div class="posts-list">
        {% for post in site.posts %}
            <div class="post-entry">
                <h2><a href="{{ post.url | relative_url }}">{{ post.title }}</a></h2>
                <span class="date">{{ post.date | date: "%B %-d, %Y" }}</span>
            </div>
        {% endfor %}
    </div>
</section>
