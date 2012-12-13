---
layout: page
title: Objective Oriented
tagline: A trip through a young entreprenuer's life.
---
{% include JB/setup %}

<div style="float:right;">
  <h3>Previous Posts</h3>
  <ul class="posts">
    {% for post in site.posts limit:5 %}
      <li><span>{{ post.date | date_to_string }}</span> &raquo; <a href="{{ BASE_PATH }}{{ post.url }}">{{ post.title }}</a></li>
    {% endfor %}
  </ul>
</div>

<p>
  Being a young entrepreneur is hard this blog is going to show the ups and downs of working for them. I will do my best to keep those involved anonymity. If you would like to share your own stories please submit a pull request with your post.
</p>

<div class="latest">
  {% assign first_post = site.posts.first %}
  <h1>{{ first_post.title }} <small>{{ first_post.description }}</small></h1>
  {{ first_post.content }}
  <a href="{{ BASE_PATH }}{{ first_post.url }}">Comment on this post.</a>
</div>