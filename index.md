---
layout: researcher
---

## About Me

I am a programmer comming from China. recently I focus on Computational Advertising, and recommendation system.


# POSTS
{% for post in site.posts %}
　　{{ post.date | date_to_string }} <a href="{{ site.baseurl }}{{ post.url }}">{{ post.title }}</a>
{% endfor %}
