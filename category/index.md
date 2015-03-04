---
layout: default
title: Category
---
<div class="page-content wc-container">
<h2>Blog Category </h2>  

{% for category in site.categories %}
    <h5>{{ category[0] }}</h5>

    <ul class="posts">
    {% for post in category[1] %}
        <li><a href="{{ post.url | prepend: site.baseurl }}">{{ post.title }}</a></li>
    {% endfor %}
    </ul>

{% endfor %}

</div>