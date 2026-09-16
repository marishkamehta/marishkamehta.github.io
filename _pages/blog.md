---
layout: default
permalink: /blog/
title: Blog
nav: true
nav_order: 3
dropdown: true
children:
  - title: Studying LLM Behavior
    permalink: /blog/
pagination:
  enabled: false
---

<div class="post">

{% assign landing_post = site.posts | where_exp: "post", "post.path contains 'controlled-experiments'" | first %}

<div class="blog-landing">
  {% if landing_post %}
    <div class="landing-post">
      <header class="post-header">
        <h1 class="post-title">{{ landing_post.title }}</h1>
      </header>
      <div class="post-content">
        {% include blog-content.liquid content=landing_post.content %}
      </div>
    </div>
  {% endif %}

{% include blog-sidebar.liquid %}
</div>

</div>
