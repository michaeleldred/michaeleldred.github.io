---
layout: home
---
<style>
    .game-list {
        list-style-type: none;
        margin: 0;
        padding:0;
        display: grid;
        width:100%;
        grid-template-columns: 310px 310px;
        gap: 20px;
        margin-bottom: 40px;
    }
    .game-list li {
        width: 310px;
        height: 100%;
    }
    .game-list a {
        cursor: pointer;
        color: black;
        width: 310px;
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

    .game-list li div {
        padding: 5px;
    }
</style>

<h1>Games</h1>
<ul class="game-list">
{% for game in site.games limit:3 %}
    <li>
        <a href="{{ game.url }}">
            <img src="/assets/infinibreak/cover_image.png" alt="{{ game.title }}"/>
        </a>
        <div>
            <h3>{{ game.title }}</h3>
            <p>{{ game.short_desc}}</p>
        </div>
    </li>
{% endfor %}
</ul>


<h1>Posts</h1>
<ul class="game-list">
{% for post in site.posts limit:3 %}
    <li>
        <a href="{{ post.url }}"><img src="{{ post.og_image }}" /></a>
        <div>
            <h3><a href="{{ post.url }}">{{ post.title }}</a></h3>
            <p>{{ post.blurb }}</p>
        </div>
    </li>
{% endfor %}
</ul>