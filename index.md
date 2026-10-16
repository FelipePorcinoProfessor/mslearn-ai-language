---
title: Develop AI Language and Speech solutions on Azure
permalink: index.html
layout: default
---

<div class="course-intro"><p>This site contains practical exercises for developing language and speech solutions with Azure AI Language, Azure Speech, Azure Translator, Microsoft Foundry and related tools.</p></div>

<div class="course-note"><strong>Note:</strong> To complete these exercises, you'll need an Azure subscription. If you don't already have one, you can sign up for an <a href="https://azure.microsoft.com/free">Azure account</a>.</div>

## Exercises {#exercises}

<div class="course-grid">
  {%- assign exercises = site.pages | where_exp: "page", "page.path contains 'Instructions/Exercises/'" | sort: "path" -%}
  {%- for activity in exercises -%}
    {%- if activity.lab.title -%}
    <article class="course-card"><h3><a href="{{ activity.url | relative_url }}">{{ activity.lab.title }}</a></h3><div class="course-meta">{%- if activity.lab.level -%}<span class="course-badge">Level {{ activity.lab.level }}</span>{%- endif -%}{%- if activity.lab.duration -%}<span class="course-badge">{{ activity.lab.duration }} minutes</span>{%- endif -%}</div>{%- if activity.lab.description -%}<p>{{ activity.lab.description }}</p>{%- endif -%}</article>
    {%- endif -%}
  {%- endfor -%}
</div>

> **More learning:** Explore the related [Microsoft Learn training path](https://learn.microsoft.com/training/paths/develop-language-solutions-azure-ai/).
