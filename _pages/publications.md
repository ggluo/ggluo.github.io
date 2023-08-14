---
layout: page
permalink: /publications/
title: Publications
description: publications by categories in reversed chronological order.
sections:
  - bibquery: "@misc|@phdthesis|@mastersthesis"
    text: "Thesis"
  - bibquery: "@arxiv"
    text: "Preprints"
  - bibquery: "@article"
    text: "Journal Articles"
  - bibquery: "@inproceedings"
    text: "Conference Proceedings and Abstracts"
nav: true
years: [2023, 2022, 2021, 2020, 2019,2017]
nav_order: 2
---
<!-- _pages/publications.md -->
An up-to-date list is available on [Google Scholar](https://scholar.google.com/citations?user=HuXdcKkAAAAJ&hl=en).

<div class="publications">

{%- for section in page.sections %}
  <a id="{{section.text}}"></a>
  <p class="bibtitle">{{section.text}}</p>
  {%- for y in page.years %}

    {%- comment -%}  Count bibliography in actual section and year {%- endcomment -%}
    {%- capture citecount -%}
    {%- bibliography_count -f {{site.scholar.bibliography}} -q {{section.bibquery}}[year={{y}}] -%}
    {%- endcapture -%}

    {%- comment -%} If exist bibliography in actual section and year, print {%- endcomment -%}
    {%- if citecount !="0" %}

      {%- comment -%} <h2 class="year">{{y}}</h2> {%- endcomment -%}
      {% bibliography -f {{site.scholar.bibliography}} -q {{section.bibquery}}[year={{y}}] %}

    {%- endif -%}

  {%- endfor %}

{%- endfor %}

</div>

