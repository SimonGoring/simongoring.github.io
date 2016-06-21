---
layout: default
title: CV
weight: 5
---

# Curriculum Vitae

A semi-curated set of research activities covering the past 10 years.  My full CV can be accessed through my [GitHub repository](https://github.com/SimonGoring/CV).

My academic research has been cited over 500 times. My non-academic publications have been used as the basis of educational modules, internal project development, and have helped spur engagement of individuals from under-represented STEM backgrounds.

My work is strongly interdisciplinary, focusing on basic plant biology, paleoecology, quantitative analysis and social systems within academia.  I contribute to the ongoing development of the [Neotoma Paleoecological Database](http://neotomadb.org/), I am a member of the [EarthCube](http://earthcube.org/) Leadership Committee, and am on the Board of Directors of the [Kids Sing Chorus](http://www.kidssing.ca/), a non-profit organization that makes a choral experience available free of charge to children in East Vancouver communities who may not otherwise have access to music education.

{% for cvs in site.cv %}
  <div class="col-lg-3 col-md-6 text-center">
    <div class="resource-box">
      <big>{{ cvs.title }}</big> &#8226; <small>{{ cvs.concept }} [<a href="{{cvs.url}}">Link</a>]</small><br><p></p>
    </div>
  </div>
{% endfor %}
