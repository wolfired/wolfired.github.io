---
layout: default.html
---
{% for post in site.posts %}
[{{ post.title }}]({{ post.url }} "{{ post.title }}")
{% endfor %}
