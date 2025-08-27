---
title: Casos
layout: default
nav_order: 2
---

# Casos

<input id="filter" type="search" placeholder="Filtra por país, sector, título…" style="padding:.6rem;width:100%;max-width:520px;margin:.5rem 0 1rem;border:1px solid #ddd;border-radius:6px;">

<table id="cases" style="width:100%;border-collapse:collapse;">
  <thead>
    <tr>
      <th style="text-align:left;padding:.6rem;border-bottom:1px solid #e5e5e5;">Título</th>
      <th style="text-align:left;padding:.6rem;border-bottom:1px solid #e5e5e5;">País</th>
      <th style="text-align:left;padding:.6rem;border-bottom:1px solid #e5e5e5;">Sector</th>
      <th style="text-align:right;padding:.6rem;border-bottom:1px solid #e5e5e5;">Año</th>
      <th style="text-align:left;padding:.6rem;border-bottom:1px solid #e5e5e5;">Resultado breve</th>
    </tr>
  </thead>
  <tbody>
  {% assign sorted = site.cases | sort: 'pais' %}
  {% for c in sorted %}
    <tr>
      <td style="padding:.6rem;border-bottom:1px solid #f1f1f1;"><a href="{{ c.url }}">{{ c.title }}</a></td>
      <td style="padding:.6rem;border-bottom:1px solid #f1f1f1;">{{ c.pais }}</td>
      <td style="padding:.6rem;border-bottom:1px solid #f1f1f1;">{% if c.sector %}{{ c.sector | join: ", " }}{% endif %}</td>
      <td style="padding:.6rem;border-bottom:1px solid #f1f1f1;text-align:right;">{% if c.año_inicio %}{{ c.año_inicio }}{% endif %}</td>
      <td style="padding:.6rem;border-bottom:1px solid #f1f1f1;">{% if c.resultados %}{{ c.resultados | first }}{% else %}—{% endif %}</td>
    </tr>
  {% endfor %}
  </tbody>
</table>

<script>
const input = document.getElementById('filter');
const rows = () => [...document.querySelectorAll('#cases tbody tr')];
input.addEventListener('input', () => {
  const q = input.value.toLowerCase().trim();
  rows().forEach(tr => {
    const text = tr.innerText.toLowerCase();
    tr.style.display = text.includes(q) ? '' : 'none';
  });
});
</script>
