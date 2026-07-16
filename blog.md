---
layout: homepage
title: Blog
description: Notes on research, engineering, large language models, and embodied intelligence by Tian Xiang.
permalink: /blog/
---

<div class="blog-index">
  <div class="blog-index-header">
    <h2>Blog</h2>
    <p>Notes on research, engineering, large language models, and embodied intelligence.</p>
  </div>

{% for post in site.posts %}
  {% include post-card.html post=post %}
{% else %}
  <p class="blog-empty">No posts yet.</p>
{% endfor %}

</div>
