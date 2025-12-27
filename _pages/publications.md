---
layout: page
permalink: /publications/
title: Publications
description: publications by categories in reversed chronological order.
sections:
  - bibquery: "@arxiv"
    text: "Preprint"
  - bibquery: "@inproceedings"
    text: "Conference Paper"
  - bibquery: "@article"
    text: "Journal Article"
  - bibquery: "@misc|@phdthesis|@mastersthesis"
    text: "Thesis"
  - bibquery: "@inabstracts"
    text: "Conference Abstract"
nav: true
years: [2025,2024, 2023, 2022, 2021, 2020, 2019,2017]
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


<div style="display: inline-block; width: 100px; height:20px; vertical-align: text-top; margin-left: 5px;">
  <script type="text/javascript" id="clustrmaps" src="//clustrmaps.com/map_v2.js?d=VmwBsxPxVDHo5jOgCLt-z4v5zFwHxmrSvQYKvcgpek4&cl=ffffff&w=a"></script>
</div>