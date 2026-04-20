---
layout: page
permalink: /publications/
title: Publications
nav: true
nav_order: 2
hide_title: true
---

<!-- _pages/publications.md -->

{% assign scholar_summary = site.data.citations.metadata.summary %}
{% if scholar_summary %}
<div class="scholar-summary" markdown="0">
  <a class="stat" href="https://scholar.google.com/citations?user={{ site.data.socials.scholar_userid }}" target="_blank" rel="noopener">
    <div class="value">{{ scholar_summary.total_citations }}</div>
    <div class="label">citations</div>
  </a>
  <a class="stat" href="https://scholar.google.com/citations?user={{ site.data.socials.scholar_userid }}" target="_blank" rel="noopener">
    <div class="value">{{ scholar_summary.h_index }}</div>
    <div class="label">h-index</div>
  </a>
  <a class="stat" href="https://scholar.google.com/citations?user={{ site.data.socials.scholar_userid }}" target="_blank" rel="noopener">
    <div class="value">{{ scholar_summary.i10_index }}</div>
    <div class="label">i10-index</div>
  </a>
</div>
{% endif %}

<!-- Bibsearch Feature -->

{% include bib_search.liquid %}

<div class="publications">

{% bibliography -q @*[preprint!=true] %}

</div>
