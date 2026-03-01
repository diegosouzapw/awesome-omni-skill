---
layout: default
title: softskill
---
<div >
  <ul class="posts">
  {% for post in site.posts %}
    {% if post.categories contains 'softskill'  %}
      
      <small>{{ post.date | date: "%b %d, %Y"}}</small> <br>
      <a href="{{ post.url }}"> {{ post.title }}</a>  <br>   
          {{ post.abstract }} <br>

    {% endif %}
  {% endfor %}
  </ul>
</div>