---
layout: page
title: Projects
permalink: /projects/
description: MR research is not rocket science, but also very cool 😃!
nav: true
nav_order: 1
horizontal: true
---
**`Overview`**
In recent years, deep learning methods have made significant advancements in fast MRI reconstruction, yielding many promising results. However, challenges are constantly rising. Supervised methods for MR reconstruction are costly because of the collection of [k-space](https://en.wikipedia.org/wiki/K-space_(magnetic_resonance_imaging)) label data and a model for one specific application is hard to apply to other applications. Apart from that, worries about the uncertainty caused by undersampling strategies and deep learning algorithms have limited their usage in clinical practice until now, which could lead to hallucinations. The application of generative image priors is one way out.

- I’m currently working on diffusion models and its application on MRI reconstruction.
- I use python, C/C++, shell and Latex. I also a developer for [BART](https://github.com/mrirecon/bart). I have spent much time with JAX/Tensorflow/Pytorch.
- I have a proven track record of developing and deploying generative models

Feel free to email me if you are interested in my research. Remote collaboration is also welcome!

<!-- pages/projects.md -->
<div class="projects">
{%- if site.enable_project_categories and page.display_categories %}
  <!-- Display categorized projects -->
  {%- for category in page.display_categories %}
  <h2 class="category">{{ category }}</h2>
  {%- assign categorized_projects = site.projects | where: "category", category -%}
  {%- assign sorted_projects = categorized_projects | sort: "importance" %}
  <!-- Generate cards for each project -->
  {% if page.horizontal -%}
  <div class="container">
    <div class="row row-cols-1">
    {%- for project in sorted_projects -%}
      {% include projects_horizontal.html %}
    {%- endfor %}
    </div>
  </div>
  {%- else -%}
  <div class="grid">
    {%- for project in sorted_projects -%}
      {% include projects.html %}
    {%- endfor %}
  </div>
  {%- endif -%}
  {% endfor %}

{%- else -%}
<!-- Display projects without categories -->
  {%- assign sorted_projects = site.projects | sort: "importance" -%}
  <!-- Generate cards for each project -->
  {% if page.horizontal -%}
  <div class="container">
    <div class="row row-cols-1">
    {%- for project in sorted_projects -%}
      {% include projects_horizontal.html %}
    {%- endfor %}
    </div>
  </div>
  {%- else -%}
  <div class="grid">
    {%- for project in sorted_projects -%}
      {% include projects.html %}
    {%- endfor %}
  </div>
  {%- endif -%}
{%- endif -%}
</div>
