---
layout: home
title: games
permalink: /games/
---

<style>
    .game-list {
        list-style-type: none;
        margin: 0;
        padding:0;
        display: block;
        width:100%;
    }
    .game-list a {
        cursor: pointer;
        color: white;
        width: 640px;
        overflow: hidden;
        display: block;
    }

    .game-list a img {
        transform: scale(1);
        transition: transform .5s ease;
    }

    .game-list a:hover img {
        transform: scale(1.04);
    }
</style>

<h1>Games</h1>
<ul class="game-list">
{% assign games = site.games | sort: "order_num" | reverse %}
{% for game in games %}
    <li class="game">
        <a href="{{ game.url }}">
            <img src="/assets/{{game.url}}/cover_image.png" alt="{{ game.title }}"/>
        </a>
        
    </li>
{% endfor %}
</ul>