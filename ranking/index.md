---
title: Ranking por impacto
layout: default
nav_order: 4
permalink: /ranking/
---

# Ranking por impacto

<div style="margin:.5rem 0 1rem;display:flex;gap:.5rem;flex-wrap:wrap">
  <input id="q" type="search" placeholder="Filtra por país, sector, título…" style="padding:.6rem;min-width:260px;border:1px solid #ddd;border-radius:6px">
  <select id="country" style="padding:.6rem;border:1px solid #ddd;border-radius:6px">
    <option value="">Todos los países</option>
    {% assign countries = site.cases | map: "pais" | uniq | sort %}
    {% for p in countries %}{% if p %}<option value="{{ p | escape }}">{{ p }}</option>{% endif %}{% endfor %}
  </select>
  <select id="sector" style="padding:.6rem;border:1px solid #ddd;border-radius:6px">
    <option value="">Todos los sectores</option>
    {% assign secs = site.cases | map: "sector" | join: "," | split: "," | uniq | sort %}
    {% for s in secs %}{% assign s2 = s | strip %}{% if s2 != "" %}<option value="{{ s2 | escape }}">{{ s2 }}</option>{% endif %}{% endfor %}
  </select>
</div>

<table id="tbl" style="width:100%;border-collapse:collapse">
  <thead>
    <tr>
      <th style="text-align:right;padding:.6rem;border-bottom:1px solid #eee">#</th>
      <th style="text-align:left;padding:.6rem;border-bottom:1px solid #eee">Caso</th>
      <th style="text-align:left;padding:.6rem;border-bottom:1px solid #eee">País</th>
      <th style="text-align:left;padding:.6rem;border-bottom:1px solid #eee">Sector</th>
      <th style="text-align:right;padding:.6rem;border-bottom:1px solid #eee">Año</th>
      <th style="text-align:right;padding:.6rem;border-bottom:1px solid #eee">Score</th>
    </tr>
  </thead>
  <tbody></tbody>
</table>

<script>
// --- Datos desde la colección ---
const RAW = [
{% for c in site.cases %}
{
  title: {{ c.title | jsonify }},
  url: "{{ c.url }}",
  pais: {{ c.pais | jsonify }},
  sector: {{ c.sector | jsonify }},
  anio: {{ c.año_inicio | default: 'null' }},
  kpis: {{ c.kpis | jsonify }},
  resultados: {{ c.resultados | jsonify }},
  score: {{ c.score | default: 'null' }}
},
{% endfor %}
];

// --- Puntuación ---
function computeScore(c) {
  if (typeof c.score === 'number') return c.score;
  const k = Array.isArray(c.kpis) ? c.kpis.length : 0;
  const r = Array.isArray(c.resultados) ? c.resultados.length : 0;
  return k*2 + r;
}

// --- Render ---
const tbody = document.querySelector('#tbl tbody');
const q = document.getElementById('q');
const country = document.getElementById('country');
const sector = document.getElementById('sector');

function render() {
  const query = (q.value || '').toLowerCase();
  const selC = country.value;
  const selS = sector.value;

  let rows = RAW.map(c => ({...c, score: computeScore(c)}));

  rows = rows.filter(c => {
    const text = [c.title, c.pais, (c.sector||[]).join(' ')].join(' ').toLowerCase();
    const okQ = query ? text.includes(query) : true;
    const okC = selC ? c.pais === selC : true;
    const okS = selS ? (c.sector||[]).includes(selS) : true;
    return okQ && okC && okS;
  });

  rows.sort((a,b) => (b.score - a.score) || ((b.anio||0)-(a.anio||0)));

  tbody.innerHTML = rows.map((c, i) => {
    const medal = i===0 ? '🥇' : i===1 ? '🥈' : i===2 ? '🥉' : (i+1);
    const sectorTxt = (c.sector||[]).join(', ');
    return `
      <tr>
        <td style="padding:.6rem;border-bottom:1px solid #f2f2f2;text-align:right">${medal}</td>
        <td style="padding:.6rem;border-bottom:1px solid #f2f2f2"><a href="${c.url}">${c.title}</a></td>
        <td style="padding:.6rem;border-bottom:1px solid #f2f2f2">${c.pais||''}</td>
        <td style="padding:.6rem;border-bottom:1px solid #f2f2f2">${sectorTxt}</td>
        <td style="padding:.6rem;border-bottom:1px solid #f2f2f2;text-align:right">${c.anio||''}</td>
        <td style="padding:.6rem;border-bottom:1px solid #f2f2f2;text-align:right"><strong>${c.score}</strong></td>
      </tr>`;
  }).join('') || `<tr><td colspan="6" style="padding:1rem;color:#666">Sin resultados con esos filtros.</td></tr>`;
}

[q, country, sector].forEach(el => el.addEventListener('input', render));
render();
</script>
