---
layout: page
permalink: /activism/
title: Activism
---

<div id="archives">
    {% for post in site.categories['Activism'] %}
    <article class="archive-item">
      <h4>
          <a href="{{ site.baseurl }}{{ post.url }}">{{post.title}}</a>
      </h4>
    </article>
    {% endfor %}
</div>
