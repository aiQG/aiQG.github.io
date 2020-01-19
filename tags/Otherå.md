---
layout: default
---

{% assign tagName = page.name | split: "å.md"  %}
{% for post in site.posts %}
    {% for tag in post.tags %}
        {% if tagName[0] == tag %}
<div id="post-li">
  <div class="header">
    <a id="urlLinkColor" href="{{ post.url }}#0"> {{ post.title }} </a>
  </div>
  <div class="content">
    <span>
      {% if post.excerpt %}{{ post.excerpt | strip_html | strip_newlines | truncate: 120 }}{% endif %}
    </span>
  </div>
  <div class="footer">
    <span> {{ post.date | date: "20%y-%m-%d" }} </span>
    <span> 
          {% if post.tags %}
          <small>
            <em>
              {{ post.tags | join: " " }}
            </em>
          </small>
           {% endif %}
           <p></p>
    </span>
  </div>
</div>
            {%break%}

        {%endif%}
    {% endfor %}
{% endfor %}
