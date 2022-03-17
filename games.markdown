---
layout: home
title: games
permalink: /games/
---

<h1>Games</h1>
<ul>
{% for game in site.games limit:3 %}
    <li><a href="{{ game.url}}">{{ game.title }}</a></li>
{% endfor %}
</ul>