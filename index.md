---
layout: default
title: StepsToSucces
nav_order: 1
---

# 🌍 StepsToSucces
**Mejores prácticas y casos de éxito gubernamentales.**  
Datos, replicabilidad e impacto global.

<div class="hero">
  <p>
    🔎 Usa la búsqueda para encontrar casos por país, sector o KPI.  
    🗺️ Explora el Mapa interactivo.  
    🏆 Revisa el Ranking.  
    ✍️ ¿Tienes un caso? <a href="/propose/">Propón uno</a>.
  </p>
</div>

---

## 🗺️ Mapa de Casos
<div id="home-map"></div>

<link rel="stylesheet" href="https://unpkg.com/leaflet@1.9.4/dist/leaflet.css"/>
<script src="https://unpkg.com/leaflet@1.9.4/dist/leaflet.js"></script>

<script>
const CASES = [
{% for c in site.cases %}
{
  title: {{ c.title | jsonify }},
  url: "{{ c.url }}",
  pais: {{ c.pais | jsonify }},
  lat: {{ c.lat | default: 'null' }},
  lng: {{ c.lng | default: 'null' }}
},
{% endfor %}
];
</script>

// ===== Mini-mapa estilo profesional =====
(function initMap(){
  const el = document.getElementById('home-map');
  if (!el) return;

  const map = L.map(el, { zoomControl:true, scrollWheelZoom:false })
               .setView([20,0], 2);

  // Tiles claros tipo Carto "Positron"
  L.tileLayer(
    'https://{s}.basemaps.cartocdn.com/light_all/{z}/{x}/{y}{r}.png',
    { attribution: '&copy; OpenStreetMap &copy; CARTO', subdomains:'abcd', maxZoom:19 }
  ).addTo(map);

  // Pin SVG (azul sobrio)
  const pin = L.divIcon({
    className: 'pin',
    html: `<svg viewBox="0 0 24 24" fill="#1f6feb" xmlns="http://www.w3.org/2000/svg">
             <path d="M12 2a7 7 0 0 0-7 7c0 5.25 7 13 7 13s7-7.75 7-13a7 7 0 0 0-7-7zm0 9.5a2.5 2.5 0 1 1 0-5 2.5 2.5 0 0 1 0 5z"/>
           </svg>`,
    iconSize: [24,24],
    iconAnchor: [12,24]
  });

  // Datos de casos (asegúrate que CASES esté definido en _data/cases.json)
  const pts = [];
  if (typeof CASES !== "undefined") {
    CASES.filter(c => c.lat && c.lng).forEach(c => {
      L.marker([c.lat, c.lng], {icon: pin})
        .addTo(map)
        .bindPopup(`<a href="${c.url}"><strong>${c.title}</strong></a><br><span class="sub">${c.pais||''}</span>`);
      pts.push([c.lat, c.lng]);
    });
    if (pts.length) map.fitBounds(pts, { padding:[20,20] });
  }
})();
</script>

---

## 📚 Explora
- 👉 [Todos los Casos](/cases/)
- 👉 [Mapa Completo](/map/)
- 👉 [Ranking de Impacto](/ranking/)
- 👉 [Propón un Caso](/propose/)

