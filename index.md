---
layout: default
title: "Global Repository of Public Policy Success Stories"
description: "Explora políticas públicas exitosas del mundo: mapa interactivo, ranking y casos curados."
use_leaflet: true
---

# Global Repository of Public Policy Success Stories
Explora iniciativas impactantes del mundo y descubre pasos al éxito en política pública.

<div class="grid">
  <!-- Mapa -->
  <section class="card">
    <h2 class="sr-only">Mapa</h2>
    <div class="card" style="margin-bottom:16px;">
      <form id="home-filters" class="chiprow" aria-label="Filtros del mapa y listado">
        <label class="chip">
          <span>País</span>
          <select id="f-country">
            <option value="">Todos</option>
          </select>
        </label>
        <label class="chip">
          <span>Sector</span>
          <select id="f-sector">
            <option value="">Todos</option>
          </select>
        </label>
        <label class="chip">
          <span>Año ≥</span>
          <input id="f-ymin" type="number" inputmode="numeric" placeholder="1900" style="width:90px">
        </label>
        <label class="chip">
          <span>Año ≤</span>
          <input id="f-ymax" type="number" inputmode="numeric" placeholder="2025" style="width:90px">
        </label>
        <label class="chip" style="flex:1 1 220px">
          <span>Buscar</span>
          <input id="f-q" type="search" placeholder="País, título, sector…" style="width:100%">
        </label>
        <button type="button" id="f-reset" class="back">Reset</button>
        <button type="button" id="f-share" class="back" title="Copiar enlace con filtros">Compartir</button>
      </form>
    </div>
    <div id="home-map" class="map" role="application" aria-label="Mapa de casos"></div>
  </section>

  <!-- Top 5 Ranking -->
  <section class="card">
    <h2>🏆 Top 5 Casos Destacados</h2>
    <ol id="rank-list" class="rank">
      {% assign sorted = site.cases | where_exp:"c","c.score != nil" | sort:"score" | reverse %}
      {% for c in sorted limit:5 %}
      <li class="rank-item">
        <a class="rank-left" href="{{ c.url | relative_url }}" aria-label="Abrir {{ c.title }}">
          <span class="medal">
            {% case forloop.index %}
              {% when 1 %}🥇{% when 2 %}🥈{% when 3 %}🥉{% else %}{{ forloop.index }}{% endcase %}
          </span>
          <div>
            <div class="country">{% if c.flag %}{{ c.flag }} {% endif %}{{ c.pais }}</div>
            <div class="small">{{ c.title }}</div>
          </div>
        </a>
        <span class="pill">{{ c.score }}</span>
      </li>
      {% endfor %}
    </ol>

    <!-- Quick insights -->
    <div class="muted small" style="margin-top:10px">
      {% assign total = site.cases | size %}
      {% assign with_coords = site.cases | where_exp:"c","c.lat and c.lng" | size %}
      Casos: <strong>{{ total }}</strong> · Con mapa: <strong>{{ with_coords }}</strong>
    </div>
  </section>
</div>

## Casos en la plataforma
<ul id="cases-list" class="cases-list">
  {% assign listed = site.cases | sort: "anio_inicio" | reverse %}
  {% for c in listed %}
  <li class="case-item" data-country="{{ c.pais | escape }}" data-sectors="{{ c.sector | join: '|' | escape }}" data-year="{{ c.anio_inicio | default: c['año_inicio'] }}" data-title="{{ c.title | escape }}">
    <a href="{{ c.url | relative_url }}">
      <span class="flag">{% if c.flag %}{{ c.flag }}{% endif %}</span>
      <span class="title">{{ c.title }}</span>
      <span class="muted">{{ c.pais }} — {{ c.anio_inicio | default: c['año_inicio'] }}</span>
    </a>
  </li>
  {% endfor %}
</ul>

<!-- MarkerCluster -->
<link rel="stylesheet" href="https://unpkg.com/leaflet.markercluster@1.5.3/dist/MarkerCluster.css">
<link rel="stylesheet" href="https://unpkg.com/leaflet.markercluster@1.5.3/dist/MarkerCluster.Default.css">
<script defer src="https://unpkg.com/leaflet.markercluster@1.5.3/dist/leaflet.markercluster.js"></script>

<script>
document.addEventListener('DOMContentLoaded', function(){
  // ---------- Datos estáticos generados por Jekyll ----------
  const CASES = [
    {% for c in site.cases %}
    {
      title: {{ c.title | jsonify }},
      country: {{ c.pais | jsonify }},
      flag: {{ c.flag | default: "" | jsonify }},
      url: {{ c.url | relative_url | jsonify }},
      lat: {{ c.lat | default: 'null' }},
      lng: {{ c.lng | default: 'null' }},
      sectors: {{ c.sector | default: empty | jsonify }},
      year: {{ c.anio_inicio | default: c['año_inicio'] | jsonify }},
      score: {{ c.score | default: 'null' }}
    }{% unless forloop.last %},{% endunless %}
    {% endfor %}
  ];

  // ---------- Utilidades ----------
  const $ = sel => document.querySelector(sel);
  const $$ = sel => Array.from(document.querySelectorAll(sel));
  const uniq = arr => Array.from(new Set(arr)).filter(Boolean).sort();
  const params = new URLSearchParams(location.search);

  // ---------- Poblar selects ----------
  const countries = uniq(CASES.map(c => c.country));
  const sectors = uniq(CASES.flatMap(c => Array.isArray(c.sectors) ? c.sectors : []));

  const $country = $('#f-country'), $sector = $('#f-sector'),
        $ymin = $('#f-ymin'), $ymax = $('#f-ymax'),
        $q = $('#f-q'), $reset = $('#f-reset'), $share = $('#f-share');

  countries.forEach(v => { const o = document.createElement('option'); o.value=v; o.textContent=v; $country.appendChild(o); });
  sectors.forEach(v => { const o = document.createElement('option'); o.value=v; o.textContent=v; $sector.appendChild(o); });

  // ---------- Init valores desde URL ----------
  if (params.has('country')) $country.value = params.get('country');
  if (params.has('sector'))  $sector.value  = params.get('sector');
  if (params.has('ymin'))    $ymin.value    = params.get('ymin');
  if (params.has('ymax'))    $ymax.value    = params.get('ymax');
  if (params.has('q'))       $q.value       = params.get('q');

  // ---------- Rango años sugerido ----------
  const years = CASES.map(c => +c.year).filter(Number.isFinite);
  const minY = years.length ? Math.min(...years) : 1900;
  const maxY = years.length ? Math.max(...years) : new Date().getFullYear();
  if (!$ymin.placeholder) $ymin.placeholder = String(minY);
  if (!$ymax.placeholder) $ymax.placeholder = String(maxY);

  // ---------- Mapa ----------
// Mapa
const map = L.map('map', { worldCopyJump: true, scrollWheelZoom: false });

L.tileLayer(
  'https://server.arcgisonline.com/ArcGIS/rest/services/Canvas/World_Light_Gray_Base/MapServer/tile/{z}/{y}/{x}', 
  {
    attribution: 'Tiles &copy; Esri &mdash; Source: Esri, DeLorme, NAVTEQ',
    maxZoom: 16
  }
).addTo(map);


  // Crear marcadores una sola vez (y los reutilizamos)
  const markers = CASES
    .filter(c => typeof c.lat === 'number' && typeof c.lng === 'number')
    .map(c => {
      const m = L.marker([c.lat, c.lng]);
      m.bindPopup(`<strong><a href="${c.url}">${c.title}</a></strong><br>${c.flag ? c.flag + ' ' : ''}${c.country}${c.year ? ' · ' + c.year : ''}${Array.isArray(c.sectors) && c.sectors.length ? '<br><span class="small muted">'+c.sectors.join(', ')+'</span>' : ''}${typeof c.score==='number' ? ' · <span class="pill">Score '+c.score+'</span>' : ''}`);
      m._case = c;
      return m;
    });

  // ---------- Filtrado global ----------
  function getFilters(){
    return {
      country: $country.value.trim(),
      sector:  $sector.value.trim(),
      ymin:    parseInt($ymin.value || $ymin.placeholder || minY, 10),
      ymax:    parseInt($ymax.value || $ymax.placeholder || maxY, 10),
      q:       $q.value.trim().toLowerCase()
    };
  }

  function match(c, f){
    if (f.country && c.country !== f.country) return false;
    if (f.sector && !(Array.isArray(c.sectors) && c.sectors.includes(f.sector))) return false;
    const y = Number(c.year);
    if (Number.isFinite(y)) {
      if (y < f.ymin || y > f.ymax) return false;
    }
    if (f.q) {
      const hay = [c.title, c.country, (c.sectors||[]).join(' ')].join(' ').toLowerCase();
      if (!hay.includes(f.q)) return false;
    }
    return true;
  }

  function applyFilters(){
    const f = getFilters();

    // URL compartible
    const sp = new URLSearchParams();
    if (f.country) sp.set('country', f.country);
    if (f.sector)  sp.set('sector', f.sector);
    if ($ymin.value) sp.set('ymin', f.ymin);
    if ($ymax.value) sp.set('ymax', f.ymax);
    if (f.q)       sp.set('q', f.q);
    const newUrl = location.pathname + (sp.toString() ? ('?' + sp.toString()) : '');
    history.replaceState(null, '', newUrl);

    // Mapa
    clusters.clearLayers();
    const mFiltered = markers.filter(m => match(m._case, f));
    mFiltered.forEach(m => clusters.addLayer(m));
    if (mFiltered.length) {
      const group = L.featureGroup(mFiltered);
      map.fitBounds(group.getBounds(), { padding:[30,30] });
    } else {
      map.setView([20,0], 2);
    }

    // Listado
    const items = $$('#cases-list .case-item');
    let visible = 0;
    items.forEach(li => {
      const c = {
        country: li.dataset.country || '',
        sectors: (li.dataset.sectors || '').split('|'),
        year:    parseInt(li.dataset.year || '0', 10),
        title:   (li.dataset.title || '')
      };
      const ok = match(c, f);
      li.style.display = ok ? '' : 'none';
      if (ok) visible++;
    });

    // Ranking (opcional: si quieres que refleje filtros, descomenta)
    // const r = $$('#rank-list .rank-item');
    // r.forEach(el => el.style.display = ''); // simple: mantenemos top 5 global
  }

  // Eventos
  [$country, $sector, $ymin, $ymax, $q].forEach(el => el.addEventListener('input', applyFilters));
  $('#home-filters').addEventListener('submit', e => e.preventDefault());
  $('#f-reset').addEventListener('click', () => { $country.value=''; $sector.value=''; $ymin.value=''; $ymax.value=''; $q.value=''; applyFilters(); });
  $('#f-share').addEventListener('click', async () => {
    try {
      await navigator.clipboard.writeText(location.href);
      const b = $('#f-share'); const old = b.textContent; b.textContent = 'Copiado ✓'; setTimeout(()=>b.textContent=old,1200);
    } catch(e) {}
  });

  // Primera carga
  applyFilters();

  // Ajuste al resize
  window.addEventListener('resize', () => setTimeout(()=>map.invalidateSize({animate:false}), 150), {passive:true});
});
</script>

<!-- JSON-LD: CollectionPage con recuento de ítems -->
<script type="application/ld+json">
{
  "@context":"https://schema.org",
  "@type":"CollectionPage",
  "name": {{ page.title | jsonify }},
  "description": {{ page.description | default: "Repositorio global de políticas públicas exitosas." | jsonify }},
  "inLanguage": "es",
  "isPartOf": { "@type":"WebSite", "name": {{ site.title | jsonify }}, "url": "{{ site.url }}" },
  "about": "Casos de estudio de políticas públicas",
  "hasPart": [
    {% for c in site.cases %}
    { "@type":"CreativeWork", "name": {{ c.title | jsonify }}, "url": "{{ site.url }}{{ c.url }}" }{% unless forloop.last %},{% endunless %}
    {% endfor %}
  ]
}
</script>
