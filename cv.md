---
layout: default
title: CV
weight: 5
---

# Curriculum Vitae

Simon J. Goring<br>
<i>Assistant Scientist</i><br>
Department of Geography<br>
University of Wisconsin - Madison<br>

<small>ORCID - [0000-0002-2700-4605](http://orcid.org/0000-0002-2700-4605#person)<br>
Google Scholar - [Simon Goring](https://scholar.google.ca/citations?user=Q3ekwgkAAAAJ&hl=en)<br>
ImpactStory - [AltMetrics](https://impactstory.org/u/0000-0002-2700-4605)</small>

My academic research has been cited over 500 times. My non-academic publications have been used as the basis of educational modules, internal project development, and have helped spur engagement of individuals from under-represented STEM backgrounds.

My work is strongly interdisciplinary, focusing on geoinformatics, paleoecology, basic plant biology, quantitative analysis and social systems within academia.  I am a project leader within the [Neotoma Paleoecological Database](http://neotomadb.org/), am a member of the [EarthCube](http://earthcube.org/) Leadership Committee and chair of the EarthCube Engagement Committee. I am on the Board of Directors of the [Kids Sing Chorus](http://www.kidssingchorus.ca/), a non-profit organization that makes a choral experience available free of charge to children in East Vancouver communities who may not otherwise have access to music education.

{% for cvs in site.cv %}
  <div class="col-lg-3 col-md-6 text-center">
    <div class="resource-box">
      <big>{{ cvs.title }}</big> &#8226; <small>{{ cvs.concept }} [<a href="{{cvs.url}}">Link</a>]</small><br><p></p>
    </div>
  </div>
{% endfor %}
