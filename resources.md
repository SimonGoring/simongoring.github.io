---
layout: default
title: Resources
---

# Resources

A collection of instructional or informational resources relating to R, web development, collaboration, or general informational content.  All of these documents are licensed under an MIT license and are free to use.  You may also consider contributing through the [GitHub repository](https://github.com/SimonGoring/simongoring.github.io) for this site.

{% for resources in site.resources %}
  <div class="col-lg-3 col-md-6 text-center">
    <div class="resource-box">
      <h3>{{ resources.title }}</h3>
      {{ resources.concept }} <a href="{{resources.url}}">[Link]</a>
      <p>
    </div>
  </div>
{% endfor %}
