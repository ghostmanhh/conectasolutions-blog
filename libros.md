---
layout: default
title: Biblioteca de IA
permalink: /libros-recursos/
---

<div class="page-header">
  <div class="container">
    <h1>Biblioteca de IA</h1>
  </div>
</div>

<div class="page-body">
  <div class="container">
    <p style="font-size:1.15rem; color:var(--ink-muted); max-width:600px; margin-bottom:2.5rem;">
      Libros de inteligencia artificial y ciencia de datos que recomendamos.
    </p>

    <ul>
      {% for libro in site.data.libros %}
      <li>
        <a href="{{ libro.url }}" target="_blank" rel="noopener"><strong>{{ libro.titulo }}</strong></a>
        — {{ libro.autor }} · <em>{{ libro.tema }}</em>
        {% if libro.gratis %}· Gratis{% endif %}
      </li>
      {% endfor %}
    </ul>
  </div>
</div>
