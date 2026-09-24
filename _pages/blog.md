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
    <h1>Studying LLM Behavior</h1>
    <p>Practical guides and experimental demonstrations, from first steps to controlled experiments.</p>
  </header>

  <section aria-labelledby="blog-series-title">
    <h2 class="blog-section-label" id="blog-series-title">The series</h2>
    <a class="blog-series-card" href="{{ '/blog/getting-started/' | relative_url }}">
      <div class="blog-series-copy">
        <span class="blog-card-kicker">Getting Started</span>
        <h3>Your first steps with LLM experiments</h3>
        <p>A practical guide for behavioral researchers. Send your first message, run a small experiment, and set up the tools you need.</p>
        <span class="blog-card-link">Explore the series <span aria-hidden="true">&rarr;</span></span>
      </div>
      <div class="blog-series-outline" aria-label="What the series covers">
        <span><b>01</b> Start with a model response</span>
        <span><b>02</b> Run your first experiment</span>
        <span><b>03</b> Find your setup and reference guides</span>
      </div>
    </a>
  </section>

  <section class="blog-articles-section" aria-labelledby="blog-articles-title">
    <div class="blog-section-heading">
      <h2 id="blog-articles-title">Experimental demonstrations</h2>
      <p>Two standalone articles. Start with either question.</p>
    </div>
    <div class="blog-article-grid">
      <a class="blog-article-card" href="{{ '/blog/2026/belief-bias/' | relative_url }}">
        <span class="blog-card-kicker">Belief bias</span>
        <h3>When Beliefs Bias LLM Reasoning</h3>
        <p>What happens when a logical conclusion conflicts with familiar beliefs?</p>
        <span class="blog-card-link">Read the article <span aria-hidden="true">&rarr;</span></span>
      </a>
      <a class="blog-article-card" href="{{ '/blog/2026/position-bias/' | relative_url }}">
        <span class="blog-card-kicker">Position bias</span>
        <h3>First or Second: How LLMs Judge Competing Responses</h3>
        <p>Does reversing two answers change which one a model prefers?</p>
        <span class="blog-card-link">Read the article <span aria-hidden="true">&rarr;</span></span>
      </a>
    </div>
  </section>
</div>
