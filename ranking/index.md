---
title: Ranking por impacto
layout: default
nav_order: 4
permalink: /ranking/
---

# 🏆 Ranking por impacto

<div class="rk-toolbar">
  <input id="rk-q" type="search" placeholder="Buscar por país, sector o título…">
  <select id="rk-country">
    <option value="">Todos los países</option>
    {% assign countries = site.cases | map: "pais" | uniq | sort %}
    {% for p in countries %}{% if p %}<option value="{{ p | escape }}">{{ p }}</option>{% endif %}{% endfor %}
  </select>
  <select id="rk-sector">
    <option value="">Todos los sectores</option>
    {% assign secs = site.cases | map: "sector" | join: "," | split: "," | uniq | sort %}
    {% for s in secs %}{% assign s2 = s | strip %}{% if s2 != "" %}<option value="{{ s2 | escape }}">{{ s2 }}</option>{% endif %}{% endfor %}
  </select>
  <select id="rk-sort">
    <option value="score">Ordenar: Score</option>
    <option value="anio">Ordenar: Año</option>
    <option value="pais">Ordenar: País</option>
    <option value="titulo">Ordenar: Título</option>
  </select>
</div>

<div id="rk-list" class="rk-grid"></div>

<script>
const CASES = [
{% for c in site.cases %}
{
  title: {{ c.title | jsonify }},
  url: "{{ c.url }}",
  pais: {{ c.pais | jsonify }},
  flag: {{ c.flag | default: "" | jsonify }},
  sector: {{ c.sector | jsonify }},
  anio: {{ c.año_inicio | default: 'null' }},
  score: {{ c.score | default: 'null' }}
},
{% endfor %}
];

// Helpers
const $ = s => document.querySelector(s);
const qEl = $('#rk-q'), cEl = $('#rk-country'), sEl = $('#rk-sector'), sortEl = $('#rk-sort');
const list = $('#rk-list');

function computeScore(c){ return (typeof c.score === 'number') ? c.score : 0; }

function medal(i){
  if (i===0) return '<div class="medal gold">1</div>';
  if (i===1) return '<div class="medal silver">2</div>';
  if (i===2) return '<div class="medal bronze">3</div>';
  return `<div class="medal">${i+1}</div>`;
}

function render(){
  const q = (qEl.value||'').toLowerCase();
  const fc = cEl.value, fs = sEl.value, sort = sortEl.value;

  let rows = CASES.map(c => ({...c, score: computeScore(c)}));

  rows = rows.filter(c=>{
    const text = [c.title, c.pais, (c.sector||[]).join(' ')].join(' ').toLowerCase();
    const okQ = q ? text.includes(q) : true;
    const okC = fc ? c.pais === fc : true;
    const okS = fs ? (c.sector||[]).includes(fs) : true;
    return okQ && okC && okS;
  });

  rows.sort((a,b)=>{
    if (sort==='anio') return (b.anio||0) - (a.anio||0);
    if (sort==='pais') return (a.pais||'').localeCompare(b.pais||'');
    if (sort==='titulo') return (a.title||'').localeCompare(b.title||'');
    // score
    return (b.score - a.score) || ((b.anio||0)-(a.anio||0));
  });

  list.innerHTML = rows.map((c,i)=>`
    <article class="rk-card">
      <div class="rk-left">
        ${medal(i)}
        <div class="rk-info">
          <h3><a href="${c.url}">${c.title}</a></h3>
          <div class="rk-meta">
            <span>${c.flag||''} ${c.pais||''}</span>
            ${c.anio ? `<span class="pill">${c.anio}</span>` : ``}
            ${(c.sector||[]).slice(0,3).map(s=>`<span class="pill">${s}</span>`).join('')}
          </div>
        </div>
      </div>
      <div class="rk-score">
        <div class="rk-score-num">${c.score ?? 0}</div>
        <div class="rk-score-label">Score</div>
      </div>
    </article>
  `).join('') || `<div class="sub" style="padding:1rem">Sin resultados con esos filtros.</div>`;
}

[qEl,cEl,sEl,sortEl].forEach(el=> el.addEventListener('input', render));
render();
</script>
