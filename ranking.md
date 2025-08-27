
---
title: Ranking
nav_order: 4
permalink: /ranking/
---

# Ranking por KPI

<div id="ranking"></div>
<script>
const cases = [
{% for c in site.cases %}
  { title: {{ c.title | jsonify }}, url: "{{ c.url }}", 
    kpis: {{ c.kpis | jsonify }}, resultados: {{ c.resultados | jsonify }}, 
    score: {{ c.score | default: 0 }} },
{% endfor %}
];

cases.forEach(c => { if(!c.score) c.score = (c.resultados||[]).length; });
cases.sort((a,b)=>b.score-a.score);

document.getElementById('ranking').innerHTML =
  '<ol>' + cases.map(c=>`<li><a href="${c.url}">${c.title}</a> — score: ${c.score}</li>`).join('') + '</ol>';
</script>
