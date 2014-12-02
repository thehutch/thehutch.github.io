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
                    {{ p.content | truncatewords:40 }}
                </div>

                <a href="{{ site.baseurl }}{{ p.url }}" class="read-more">Read More</a>
            </article>
        {% endif %}
    {% endfor %}
</div>
