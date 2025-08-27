
---
title: Mapa
nav_order: 3
permalink: /map/
---

<div id="map" style="height: 560px;"></div>
<link rel="stylesheet" href="https://unpkg.com/leaflet@1.9.4/dist/leaflet.css"/>
<script src="https://unpkg.com/leaflet@1.9.4/dist/leaflet.js"></script>

<script>
const cases = [
{% for c in site.cases %}
  { title: {{ c.title | jsonify }}, country: {{ c.pais | jsonify }}, url: "{{ c.url }}",
    lat: {{ c.lat | default: 'null' }}, lng: {{ c.lng | default: 'null' }} },
{% endfor %}
];

const map = L.map('map', { worldCopyJump: true }).setView([20,0], 2);
L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png', { maxZoom: 6 }).addTo(map);

cases.filter(x=>x.lat && x.lng).forEach(c=>{
  L.marker([c.lat, c.lng]).addTo(map).bindPopup(`<a href="${c.url}">${c.title}</a><br>${c.country}`);
});
</script>
