---
layout: researcher
---

## About Me

I come from China and now works in Singapore. I am a programmer, focus on Computational Advertising, and Recommendation System.


# POSTS
{% for post in site.posts %}
　　{{ post.date | date_to_string }} <a href="{{ site.baseurl }}{{ post.url }}">{{ post.title }}</a>
{% endfor %}
