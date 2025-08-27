---
title: "Ranking por impacto"
layout: default
permalink: /ranking/
description: "Ranking dinámico de casos por impacto con filtros, orden, paginación y exportación."
---

# 🏆 Ranking por impacto

<div class="card rk-toolbar" role="region" aria-label="Controles de ranking">
  <div class="rk-controls">
    <input id="rk-q" type="search" placeholder="Buscar por país, sector o título…" aria-label="Buscar">
    <select id="rk-country" aria-label="Filtrar por país">
      <option value="">Todos los países</option>
      {% assign countries = site.cases | map: "pais" | uniq | sort %}
      {% for p in countries %}{% if p %}<option value="{{ p | escape }}">{{ p }}</option>{% endif %}{% endfor %}
    </select>
    <select id="rk-sector" aria-label="Filtrar por sector">
      <option value="">Todos los sectores</option>
      {% assign secs = site.cases | map: "sector" | join: "," | split: "," | uniq | sort %}
      {% for s in secs %}{% assign s2 = s | strip %}{% if s2 != "" %}<option value="{{ s2 | escape }}">{{ s2 }}</option>{% endif %}{% endfor %}
    </select>

    <input id="rk-ymin" type="number" inputmode="numeric" placeholder="Año ≥" aria-label="Año mínimo" style="width:110px">
    <input id="rk-ymax" type="number" inputmode="numeric" placeholder="Año ≤" aria-label="Año máximo" style="width:110px">
    <input id="rk-minscore" type="number" inputmode="numeric" placeholder="Score ≥" aria-label="Score mínimo" style="width:110px">

    <select id="rk-sort" aria-label="Ordenar">
      <option value="score">Ordenar: Score</option>
      <option value="anio">Ordenar: Año</option>
      <option value="pais">Ordenar: País</option>
      <option value="titulo">Ordenar: Título</option>
    </select>

    <select id="rk-view" aria-label="Vista">
      <option value="detail">Vista: Detallada</option>
      <option value="compact">Vista: Compacta</option>
    </select>

    <button id="rk-reset" class="back" type="button" title="Reiniciar filtros">Reset</button>
    <button id="rk-share" class="back" type="button" title="Copiar enlace con filtros">Compartir</button>
    <button id="rk-export" class="back" type="button" title="Exportar CSV">Exportar</button>
  </div>

  <div class="rk-stats muted small" aria-live="polite">
    <span id="rk-count"></span> · Promedio score: <strong id="rk-avg">—</strong>
  </div>
</div>

<div id="rk-list" class="rk-grid" role="list"></div>

<div class="rk-pager">
  <button id="rk-prev" class="back" type="button" disabled>← Anterior</button>
  <span id="rk-page-indicator" class="muted"></span>
  <button id="rk-next" class="back" type="button" disabled>Siguiente →</button>
</div>

<script>
/* ===== Datos de Jekyll ===== */
const CASES = [
{% for c in site.cases %}
{
  title: {{ c.title | jsonify }},
  url: {{ c.url | relative_url | jsonify }},
  pais: {{ c.pais | jsonify }},
  flag: {{ c.flag | default: "" | jsonify }},
  sector: {{ c.sector | jsonify }},
  anio: {{ c.anio_inicio | default: c['año_inicio'] | jsonify }},
  score: {{ c.score | default: 'null' }}
}{% unless forloop.last %},{% endunless %}
{% endfor %}
];

/* ===== Utilidades ===== */
const $ = s => document.querySelector(s);
const $$ = s => Array.from(document.querySelectorAll(s));
const params = new URLSearchParams(location.search);
const clamp = (n, a, b) => Math.max(a, Math.min(b, n));
const toNum = v => (v===null || v===undefined || v==="" || isNaN(+v)) ? null : +v;

/* ===== Elementos ===== */
const qEl = $('#rk-q'), cEl = $('#rk-country'), sEl = $('#rk-sector'),
      yminEl = $('#rk-ymin'), ymaxEl = $('#rk-ymax'), minscoreEl = $('#rk-minscore'),
      sortEl = $('#rk-sort'), viewEl = $('#rk-view'),
      resetEl = $('#rk-reset'), shareEl = $('#rk-share'), exportEl = $('#rk-export'),
      list = $('#rk-list'),
      prevEl = $('#rk-prev'), nextEl = $('#rk-next'), pageInd = $('#rk-page-indicator'),
      countEl = $('#rk-count'), avgEl = $('#rk-avg');

/* ===== Preferencias iniciales desde URL ===== */
if (params.has('q')) qEl.value = params.get('q');
if (params.has('country')) cEl.value = params.get('country');
if (params.has('sector')) sEl.value = params.get('sector');
if (params.has('ymin')) yminEl.value = params.get('ymin');
if (params.has('ymax')) ymaxEl.value = params.get('ymax');
if (params.has('minscore')) minscoreEl.value = params.get('minscore');
if (params.has('sort')) sortEl.value = params.get('sort');
if (params.has('view')) viewEl.value = params.get('view');

/* ===== Paginación ===== */
let PAGE = clamp(toNum(params.get('page')) ?? 1, 1, 9999);
const PER_PAGE = 20;

/* ===== Núcleo de filtrado/orden ===== */
const computeScore = c => (typeof c.score === 'number') ? c.score : 0;

function getFilters(){
  return {
    q: (qEl.value || '').toLowerCase().trim(),
    country: cEl.value || '',
    sector: sEl.value || '',
    ymin: toNum(yminEl.value),
    ymax: toNum(ymaxEl.value),
    minscore: toNum(minscoreEl.value),
    sort: sortEl.value,
    view: viewEl.value
  };
}

function match(c, f){
  if (f.country && c.pais !== f.country) return false;
  if (f.sector && !((c.sector||[]).includes(f.sector))) return false;
  if (f.minscore !== null && computeScore(c) < f.minscore) return false;

  if (f.ymin !== null) { const y = toNum(c.anio); if (y !== null && y < f.ymin) return false; }
  if (f.ymax !== null) { const y = toNum(c.anio); if (y !== null && y > f.ymax) return false; }

  if (f.q){
    const hay = [c.title, c.pais, (c.sector||[]).join(' ')].join(' ').toLowerCase();
    if (!hay.includes(f.q)) return false;
  }
  return true;
}

function order(rows, sort){
  if (sort==='anio') return rows.sort((a,b)=> (toNum(b.anio)||0)-(toNum(a.anio)||0));
  if (sort==='pais') return rows.sort((a,b)=> (a.pais||'').localeCompare(b.pais||''));
  if (sort==='titulo') return rows.sort((a,b)=> (a.title||'').localeCompare(b.title||''));
  // score (tie-break por año)
  return rows.sort((a,b)=> (computeScore(b)-computeScore(a)) || ((toNum(b.anio)||0)-(toNum(a.anio)||0)));
}

/* ===== Render ===== */
function bar(score){
  const s = clamp(score||0, 0, 100);
  return `<div class="rk-bar" aria-hidden="true"><span style="width:${s}%"></span></div>`;
}

function medal(i){
  if (i===0) return '<span class="medal">🥇</span>';
  if (i===1) return '<span class="medal">🥈</span>';
  if (i===2) return '<span class="medal">🥉</span>';
  return `<span class="medal">${i+1}</span>`;
}

function card(c, i, view){
  const score = computeScore(c);
  const pills = (c.sector||[]).slice(0,3).map(s=>`<span class="pill">${s}</span>`).join('');
  const yearPill = c.anio ? `<span class="pill">${c.anio}</span>` : '';
  const country = `${c.flag||''} ${c.pais||''}`.trim();
  if (view === 'compact'){
    return `
    <article class="rk-card compact" role="listitem">
      <a class="rk-left" href="${c.url}">
        ${medal(i)}
        <div class="rk-info">
          <h3>${c.title}</h3>
          <div class="rk-meta">
            <span>${country}</span>${yearPill}${pills}
          </div>
        </div>
      </a>
      <div class="rk-score">
        <div class="rk-score-num">${score}</div>
        ${bar(score)}
      </div>
    </article>`;
  }
  // Detail
  return `
  <article class="rk-card" role="listitem">
    <div class="rk-left">
      ${medal(i)}
      <div class="rk-info">
        <h3><a href="${c.url}">${c.title}</a></h3>
        <div class="rk-meta">
          <span>${country}</span>${yearPill}${pills}
        </div>
      </div>
    </div>
    <div class="rk-score">
      <div class="rk-score-num">${score}</div>
      <div class="rk-score-label">Score</div>
      ${bar(score)}
    </div>
  </article>`;
}

function updateURL(f){
  const sp = new URLSearchParams();
  if (f.q) sp.set('q', f.q);
  if (f.country) sp.set('country', f.country);
  if (f.sector) sp.set('sector', f.sector);
  if (f.ymin !== null) sp.set('ymin', f.ymin);
  if (f.ymax !== null) sp.set('ymax', f.ymax);
  if (f.minscore !== null) sp.set('minscore', f.minscore);
  if (f.sort && f.sort!=='score') sp.set('sort', f.sort);
  if (f.view && f.view!=='detail') sp.set('view', f.view);
  sp.set('page', PAGE);
  history.replaceState(null, '', location.pathname + (sp.toString() ? '?' + sp.toString() : ''));
}

function render(){
  const f = getFilters();

  // Filtrar y ordenar
  let rows = CASES.map(c => ({...c})).filter(c => match(c, f));
  rows = order(rows, f.sort);

  // Stats
  const n = rows.length;
  const avg = n ? (rows.reduce((acc,c)=>acc+computeScore(c),0)/n) : 0;
  countEl.textContent = `Resultados: ${n}`;
  avgEl.textContent = n ? avg.toFixed(1) : '—';

  // Paginación
  const totalPages = Math.max(1, Math.ceil(n / PER_PAGE));
  PAGE = clamp(PAGE, 1, totalPages);
  const start = (PAGE-1) * PER_PAGE;
  const pageRows = rows.slice(start, start + PER_PAGE);

  // Render items
  list.innerHTML = pageRows.map((c, i) => card(c, start + i, f.view)).join('') || `<div class="sub" style="padding:1rem">Sin resultados con esos filtros.</div>`;
  pageInd.textContent = `Página ${PAGE} de ${totalPages}`;
  prevEl.disabled = PAGE<=1; nextEl.disabled = PAGE>=totalPages;

  updateURL(f);
}

/* ===== Eventos ===== */
[qEl, cEl, sEl, yminEl, ymaxEl, minscoreEl, sortEl, viewEl].forEach(el => {
  el.addEventListener('input', () => { PAGE = 1; render(); });
});

// Paginación
prevEl.addEventListener('click', () => { PAGE = Math.max(1, PAGE-1); render(); });
nextEl.addEventListener('click', () => { PAGE = PAGE+1; render(); });

// Reset / Share / Export
$('#rk-reset').addEventListener('click', () => {
  qEl.value = cEl.value = sEl.value = '';
  yminEl.value = ymaxEl.value = minscoreEl.value = '';
  sortEl.value = 'score'; viewEl.value = 'detail'; PAGE = 1; render();
});
$('#rk-share').addEventListener('click', async () => {
  try { await navigator.clipboard.writeText(location.href);
        const b = $('#rk-share'); const t=b.textContent; b.textContent='Copiado ✓'; setTimeout(()=>b.textContent=t,1200);
  } catch(e){}
});
$('#rk-export').addEventListener('click', () => {
  // Exporta el subconjunto filtrado (todas las páginas) a CSV
  const f = getFilters();
  const rows = order(CASES.filter(c => match(c,f)), f.sort);

  const head = ['rank','title','pais','sector','anio','score','url'];
  const csv = [head.join(',')].concat(
    rows.map((c,i)=>{
      const sector = (c.sector||[]).join('|').replace(/"/g,'""');
      return [i+1, `"${(c.title||'').replace(/"/g,'""')}"`, `"${(c.pais||'').replace(/"/g,'""')}"`,
              `"${sector}"`, c.anio||'', computeScore(c), `"${location.origin}${c.url}"`].join(',');
    })
  ).join('\n');

  const blob = new Blob([csv], {type:'text/csv;charset=utf-8;'});
  const url = URL.createObjectURL(blob);
  const a = document.createElement('a');
  a.href = url; a.download = 'ranking.csv'; a.click();
  URL.revokeObjectURL(url);
});

/* ===== Kickoff ===== */
render();
</script>
