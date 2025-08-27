---
title: Ranking
layout: default
nav_order: 4
---

# Ranking por impacto

<div id="ranking"></div>

<script>
const cases = [
{% for c in site.cases %}
  {
    title: {{ c.title | jsonify }},
    url: "{{ c.url }}",
    resultados: {{ c.resultados | jsonify }},
    score: {{ c.score | default: 'null' }}
  },
{% endfor %}
];

// Si no hay score en YAML, usa cantidad de resultados como score
cases.forEach(c => { if (typeof c.score !== 'number') c.score = (c.resultados || []).length; });
cases.sort((a,b) => b.score - a.score);

document.getElementById('ranking').innerHTML =
  '<ol>' + cases.map(c => `<li><a href="${c.url}">${c.title}</a> — score: ${c.score}</li>`).join('') + '</ol>';
</script>
