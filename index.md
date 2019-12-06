---
layout: default
---


{% for post in site.posts %}

<div id="post-li">
  <div class="header">
    <a href="{{ post.url }}#0"> {{ post.title }} </a>
  </div>
  <div class="content">
    <span>
      {% if post.excerpt %}{{ post.excerpt | strip_html | strip_newlines | truncate: 120 }}{% endif %}
    </span>
  </div>
  <div class="footer">
    <span> {{ post.date | date: "20%y-%m-%d" }} </span>
    <span> {% if post.tags %}
          <small>
            <em>
              {{ post.tags | join: " " }}
            </em>
          </small>
           {% endif %}
    </span>
  </div>
</div>

{% endfor %}
