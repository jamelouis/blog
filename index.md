---
layout: post
title: Blog - Categories
---
 <section>
{% for topic in site.data.topics %}
    <div class="topic">
        <a href="topics/{{ topic.name }}.html">
            <h2 class="topic-title">{{ topic.name }}
            {% for cate in site.categories %}
                {% if topic.name == cate[0] %}
                    <span class="articles-count">{{ cate[1] | size }}篇</span>
                {% endif %}
            {% endfor %}
            </h2>
        </a>
        <p class="clear-float">{{ topic.desc }}</p>   
    </div>
{% endfor %}
</section>
