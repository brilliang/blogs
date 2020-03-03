---
layout: researcher
---

## About Me

I'm from China and currently working in London.


# POSTS
{% for post in site.posts %}
　　{{ post.date | date_to_string }} <a href="{{ site.baseurl }}{{ post.url }}">{{ post.title }}</a>
{% endfor %}
