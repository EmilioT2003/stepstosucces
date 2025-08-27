---
layout: default
title: "Mapa"
permalink: /map/
nav_order: 3
---

<div id="map" class="map"></div>

<script>
document.addEventListener('DOMContentLoaded', function () {
  const cases = [
    {% for c in site.cases %}
    {
      title: {{ c.title | jsonify }},
      country: {{ c.pais | jsonify }},
      url: {{ c.url | relative_url | jsonify }},
      lat: {{ c.lat | default: 'null' }},
      lng: {{ c.lng | default: 'null' }}
    }{% unless forloop.last %},{% endunless %}
    {% endfor %}
  ];

  const map = L.map('map', { worldCopyJump: true, scrollWheelZoom: false })
               .setView([20, 0], 2);

  L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png', {
    attribution: '© OpenStreetMap'
  }).addTo(map);

  cases
    .filter(c => typeof c.lat === 'number' && typeof c.lng === 'number')
    .forEach(c => {
      L.marker([c.lat, c.lng]).addTo(map)
        .bindPopup(`<a href="${c.url}">${c.title}</a><br>${c.country}`);
    });
});
</script>
