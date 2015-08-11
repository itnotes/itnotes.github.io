---
layout: page
title: Tags
desc: "List of tags"
permalink: /tag/
---
{% assign tags = site.tags | sort %}
{% for tag in tags %}<span class="site-tag">
    <a class="btn btn-primary" href="/tag/#{{ tag | first | slugify }}">
            {{ tag[0] | replace:'-', ' ' }} <span class="badge">{{ tag | last | size }}</span>
    </a>
</span>{% endfor %}

---

{% for tag in site.tags %}
  {% assign t = tag | first %}
  {% assign posts = tag | last %}

<h4><a name="{{t | downcase | replace:" ","-" }}"></a><a class="internal" href="/tag/#{{t | downcase | replace:" ","-" }}">{{ t | downcase }}</a></h4>
<ul>
{% for post in posts %}
  {% if post.tags contains t %}
  <li>
    <a href="{{ post.url }}">{{ post.title }}</a>
    <span class="date">{{ post.date | date: "%B %-d, %Y"  }}</span>
  </li>
  {% endif %}
{% endfor %}
</ul>

---

{% endfor %}