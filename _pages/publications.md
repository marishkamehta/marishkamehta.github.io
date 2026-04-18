---
layout: page
permalink: /publications/
title: publications
nav: true
nav_order: 2
---

<!-- _pages/publications.md -->

<!-- Bibsearch Feature -->

{% include bib_search.liquid %}

<div class="publications">

<h2>Preprints</h2>

{% bibliography -q @*[preprint=true] %}

<h2>Published</h2>

{% bibliography -q @*[preprint!=true] %}

</div>
