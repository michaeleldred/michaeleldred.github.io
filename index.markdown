---
layout: home
---


<h1>Games</h1>
<ul>
{% for game in site.games limit:3 %}
    <li><a href="{{ game.url}}">{{ game.title }}</a></li>
{% endfor %}
</ul>


<h1>Posts</h1>
<ul>
{% for post in site.posts limit:3 %}
    <li><a href="{{ post.url}}">{{ post.title }}</a></li>
{% endfor %}
</ul>