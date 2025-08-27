---
layout: default
title: "Mapa"
description: "Explora casos de políticas públicas en el mapa, filtra por país, sector y año."
permalink: /map/
use_leaflet: true
nav_order: 3
---

# Mapa de casos
<div class="card" style="margin-bottom:16px;">
  <form id="map-filters" class="chiprow" aria-label="Filtros del mapa">
    <label class="chip">
      <span>País</span>
      <select id="filter-country">
        <option value="">Todos</option>
      </select>
    </label>
    <label class="chip">
      <span>Sector</span>
      <select id="filter-sector">
        <option value="">Todos</option>
      </select>
    </label>
    <label class="chip">
      <span>Año ≥</span>
      <input id="filter-year-min" type="number" inputmode="numeric" placeholder="1900" style="width:90px">
    </label>
    <label class="chip">
      <span>Año ≤</span>
      <input id="filter-year-max" type="number" inputmode="numeric" placeholder="2025" style="width:90px">
    </label>
    <button type="button" id="filter-reset" class="back">Reset</button>
  </form>
</div>

<div id="map" class="map" role="application" aria-label="Mapa de casos"></div>

<!-- MarkerCluster (estilos + script) -->
<link rel="stylesheet" href="https://unpkg.com/leaflet.markercluster@1.5.3/dist/MarkerCluster.css">
<link rel="stylesheet" href="https://unpkg.com/leaflet.markercluster@1.5.3/dist/MarkerCluster.Default.css">
<script defer src="https://unpkg.com/leaflet.markercluster@1.5.3/dist/leaflet.markercluster.js"></script>

<script>
document.addEventListener('DOMContentLoaded', function () {
  // Datos desde Jekyll
  const cases = [
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
  ].filter(c => typeof c.lat === 'number' && typeof c.lng === 'number');

  // Poblar selects (únicos ordenados)
  const countries = Array.from(new Set(cases.map(c => c.country).filter(Boolean))).sort();
  const sectors = Array.from(new Set(cases.flatMap(c => Array.isArray(c.sectors) ? c.sectors : []).filter(Boolean))).sort();

  const $country = document.getElementById('filter-country');
  const $sector  = document.getElementById('filter-sector');
  const $ymin    = document.getElementById('filter-year-min');
  const $ymax    = document.getElementById('filter-year-max');
  const $reset   = document.getElementById('filter-reset');

  countries.forEach(v => { const o = document.createElement('option'); o.value=v; o.textContent=v; $country.appendChild(o); });
  sectors.forEach(v => { const o = document.createElement('option'); o.value=v; o.textContent=v; $sector.appendChild(o); });

  // Determinar rango de años sugerido
  const years = cases.map(c => +c.year).filter(n => Number.isFinite(n));
  const minY = years.length ? Math.min(...years) : 1900;
  const maxY = years.length ? Math.max(...years) : new Date().getFullYear();
  $ymin.placeholder = String(minY);
  $ymax.placeholder = String(maxY);

  // Mapa
  const map = L.map('map', { worldCopyJump: true, scrollWheelZoom: false });
  const tiles = L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png', { attribution: '© OpenStreetMap' });
  tiles.addTo(map);

  // Cluster layer
  const clusters = L.markerClusterGroup({ showCoverageOnHover: false, maxClusterRadius: 50 });

  // Render inicial
  function popupHtml(c){
    const flag = c.flag ? c.flag + ' ' : '';
    const yr = (c.year ?? '').toString();
    const sec = (Array.isArray(c.sectors) && c.sectors.length) ? c.sectors.join(', ') : '';
    const score = (typeof c.score === 'number') ? ` · <span class="pill">Score ${c.score}</span>` : '';
    return `<strong><a href="${c.url}">${c.title}</a></strong><br>${flag}${c.country}${yr ? ' · ' + yr : ''}${sec ? '<br><span class="small muted">'+sec+'</span>' : ''}${score}`;
  }

  function applyFilters() {
    const fc = $country.value;
    const fs = $sector.value;
    const ymin = parseInt($ymin.value || $ymin.placeholder, 10);
    const ymax = parseInt($ymax.value || $ymax.placeholder, 10);

    clusters.clearLayers();

    const filtered = cases.filter(c => {
      if (fc && c.country !== fc) return false;
      if (fs && !(Array.isArray(c.sectors) && c.sectors.includes(fs))) return false;
      const y = Number(c.year);
      if (Number.isFinite(y)) {
        if (y < ymin || y > ymax) return false;
      }
      return true;
    });

    filtered.forEach(c => {
      const m = L.marker([c.lat, c.lng]).bindPopup(popupHtml(c));
      clusters.addLayer(m);
    });

    if (!map.hasLayer(clusters)) clusters.addTo(map);

    if (filtered.length) {
      const group = L.featureGroup(filtered.map(c => L.marker([c.lat, c.lng])));
      map.fitBounds(group.getBounds(), { padding: [30, 30] });
    } else {
      // Vista global por defecto
      map.setView([20, 0], 2);
    }
  }

  // Eventos
  [$country, $sector, $ymin, $ymax].forEach(el => el.addEventListener('change', applyFilters));
  $reset.addEventListener('click', () => {
    $country.value = '';
    $sector.value = '';
    $ymin.value = '';
    $ymax.value = '';
    applyFilters();
  });

  // Primera carga
  applyFilters();

  // Responsivo: reajustar al mostrar teclado/resize
  window.addEventListener('resize', () => {
    setTimeout(() => map.invalidateSize({ animate: false }), 150);
  }, { passive: true });
});
</script>
