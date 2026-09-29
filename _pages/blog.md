---
layout: default
permalink: /blog/
title: Blog
nav: true
nav_order: 3
description: "Guides and experimental demonstrations for studying LLM behavior."
pagination:
  enabled: false
---

<div class="blog-index">
  <header class="blog-index-header">
    <h1>Blog</h1>
  </header>

{% assign featured_slugs = 'controlled-experiments,belief-bias,position-bias' | split: ',' %}
  <div class="blog-article-grid" aria-label="Blog articles">
    {% for slug in featured_slugs %}
      {% assign entry = site.posts | where: 'slug', slug | first %}
      <article class="blog-article-card">
        <h2><a href="{{ entry.url | relative_url }}">{{ entry.card_title | default: entry.title }}</a></h2>
        <p>{{ entry.card_description | default: entry.description }}</p>
        {% include blog-tags.liquid tags=entry.tags %}
        <a class="blog-card-link" href="{{ entry.url | relative_url }}" aria-label="Read {{ entry.card_title | default: entry.title | escape }}">Read article <span aria-hidden="true">&rarr;</span></a>
      </article>
    {% endfor %}
  </div>
</div>
