---
layout: default
title: "Global Repository of Public Policy Success Stories"
---

# Global Repository of Public Policy Success Stories
Explora iniciativas impactantes del mundo y descubre pasos al éxito en política pública.

<div class="grid">
  <section class="card">
    <div id="home-map" class="map"></div>
  </section>

  <section class="card">
    <h2>Ranking</h2>
    <div class="rank">
      <div class="rank-item">
        <div>🥇 <strong>Denmark</strong><div class="muted">Renewable Energy Program</div></div>
        <div class="muted">↑ 20%</div>
      </div>
      <div class="rank-item">
        <div>🥈 <strong>Estonia</strong><div class="muted">Digital Government Services</div></div>
        <div class="pill">83.1</div>
      </div>
      <div class="rank-item">
        <div>🥉 <strong>Italy</strong><div class="muted">Universal Basic Income</div></div>
        <div class="pill">83.1</div>
      </div>
    </div>
  </section>
</div>

## Casos
<ul class="cases-list">
{% for c in site.cases %}
  <li class="case-item">
    <a href="{{ c.url | relative_url }}">
      <span class="flag">{{ c.flag }}</span>
      <span class="title">{{ c.title }}</span>
      <span class="muted">{{ c.pais }} — {{ c.anio_inicio | default: c['año_inicio'] }}</span>
    </a>
  </li>
{% endfor %}
</ul>

<script>
document.addEventListener('DOMContentLoaded', function(){
  const m = L.map('home-map',{scrollWheelZoom:false}).setView([20,0], 2);
  L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png',{attribution:'© OpenStreetMap'}).addTo(m);
  {% for c in site.cases %}
    {% if c.lat and c.lng %}
      L.marker([{{ c.lat }}, {{ c.lng }}]).addTo(m).bindPopup("{{ c.title | escape }}");
    {% endif %}
  {% endfor %}
});
</script>
