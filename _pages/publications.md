---
layout: page
permalink: /publications/
title: publications
description: Papers.
years: [2023,2022,2021,2020]
nav: true
nav_order: 1
---

see also [Google scholar](https://scholar.google.co.uk/citations?hl=en&user=CdpLhlgAAAAJ), * denote equal contribution.


<!-- _pages/publications.md -->
<div class="publications">

{%- for y in page.years %}
  <h2 class="year">{{y}}</h2>
  {% bibliography -f papers -q @*[year={{y}}]* %}
{% endfor %}

</div>
