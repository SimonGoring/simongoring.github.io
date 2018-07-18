---
layout: default
title: Resources
weight: 4
---

# Resources

<div>
	<object type="image/svg+xml" data="../images/commit_sjg.svg" style="float:right;width:18%; padding:5px;border-color:gray;border-style:solid;border-width:0.5px;margin-left:8px;">
	  Commit to the cloud!
	  <!-- fallback image in CSS -->
	</object>
	<span style="text-align:full;">
	A collection of instructional or informational resources relating to R, web development, collaboration, or general informational content.  All of these documents are licensed under an MIT license and are free to use.  You may also consider contributing through <a href="https://github.com/SimonGoring/simongoring.github.io">my homepage's GitHub repository</a>.  Many of these linked documents also have their own GitHub repositories, indicated within the document and you are always welcome to contribute there too.</span>
</div>

<h2>Pages</h2>
<hr style="color:gray;margin-bottom:25px">

{% assign items = site.resources | sort: 'res_class' | sort: 'title' %}

{% assign classes = "Development,GitHub,Misc,R" | split: "," %}

{% for class in classes %}

  <h3 style="display:inline;">{{ class }}</h3>

  {% for resources in items %}

	  {% if resources.res_class  == class %}

<div class="col-lg-3 col-md-6 text-center">
	<div class="resource-box">
		{{ resources.title }}  [<a href="{{resources.url}}">Link</a>]<br>
	  <span style = "display:inline-block;width:80%;color:gray;">{{ resources.concept }}</span><span style="float:right;color:green;"><small>{{ resources.res_class }}</small></span>
	  <br><p></p>
	</div>
</div>
	  {% endif %}
  {% endfor %}
{% endfor %}
