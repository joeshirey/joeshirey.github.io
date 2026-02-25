---
layout: default
title: Joe Shirey | Writing
pagination: 
  enabled: true
---



<section class="container">
    <div class="posts-list">
        {% for post in paginator.posts %}
            <div class="post-entry">
                <div class="post-info">
                    <h2><a href="{{ post.url | relative_url }}">{{ post.title }}</a></h2>
                    {% if post.excerpt %}
                        <p class="excerpt">{{ post.excerpt | strip_html | truncatewords: 30 }}</p>
                        <a href="{{ post.url | relative_url }}" class="read-more">Read More &rarr;</a>
                    {% endif %}
                </div>
                <div class="post-meta">
                    <span class="date">{{ post.date | date: "%B %-d, %Y" }}</span>
                    {% if post.tags.size > 0 %}
                        <div class="tags">
                            {% for tag in post.tags %}
                                {% assign tag_slug = tag | slugify %}
                                <a href="{{ '/tag/' | append: tag_slug | append: '/' | relative_url }}" class="tag">{{ tag }}</a>
                            {% endfor %}
                        </div>
                    {% endif %}
                </div>
            </div>
        {% endfor %}

        <!-- Pagination Links -->
        {% if paginator.total_pages > 1 %}
        <div class="pagination">
          {% if paginator.previous_page %}
            <a href="{{ paginator.previous_page_path | relative_url }}" class="previous">&laquo; Prev</a>
          {% else %}
            <span class="previous disabled">&laquo; Prev</span>
          {% endif %}
          
          <span class="page_number">
            Page {{ paginator.page }} of {{ paginator.total_pages }}
          </span>

          {% if paginator.next_page %}
            <a href="{{ paginator.next_page_path | relative_url }}" class="next">Next &raquo;</a>
          {% else %}
            <span class="next disabled">Next &raquo;</span>
          {% endif %}
        </div>
        {% endif %}
    </div>
</section>
