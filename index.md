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
    {% assign top = site.cases | where_exp:"c","c.score != nil" | sort:"score" | reverse %}
    {% for c in top limit:5 %}
      <a class="rank-item" href="{{ c.url | relative_url }}">
        <div class="rank-left">
          <span class="medal">
            {% case forloop.index %}
              {% when 1 %}🥇{% when 2 %}🥈{% when 3 %}🥉{% else %}{{ forloop.index }}{% endcase %}
          </span>
          <div>
            <div class="country">{{ c.pais }}{% if c.flag %} {{ c.flag }}{% endif %}</div>
            <div class="small">{{ c.title }}</div>
          </div>
        </div>
        <span class="pill">{{ c.score }}</span>
      </a>
    {% endfor %}
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
