---
layout: default
permalink: /projects/
---
<div class="posts">
    {% for p in site.posts %}
        {% if p.category == "project" %}
            <article class="post">
                <h1>
                    <a href="{{ site.baseurl }}{{ p.url }}">{{ p.title }}</a>
                </h1>
                <div class="entry">
                    {{ p.description }}
                </div>
            </article>
        {% endif %}
    {% endfor %}
</div>
