---
layout: home
---

{% for post in site.posts %}
  <img src="{% post.feature_img %}" />
{% endfor %}
