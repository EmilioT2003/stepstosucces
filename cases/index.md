---
title: Casos
layout: default
nav_order: 2
---

# Casos

| Título | País | Sector | Año | Resultados (breve) |
|---|---|---|---:|---|
{% assign sorted = site.cases | sort: 'pais' %}
{% for c in sorted %}
| [{{ c.title }}]({{ c.url }}) | {{ c.pais }} | {{ c.sector | join: ", " }} | {{ c.año_inicio }} | {{ c.resultados | first | default: "—" }} |
{% endfor %}
