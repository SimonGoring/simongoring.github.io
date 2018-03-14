---
layout: default
weight: 1
---

<h1>Solving Wicked Problems</h1>

<img src="images/Wicked_East.png" style="float:right; margin-left:10px" title="The Wicked Witch of the East as pictured in The Tin Woodman of Oz by L. Frank Baum.">
We are faced with "wicked problems"<sup><small>1</small></sup> requiring us to find <b>cross-disciplinary answers</b> to solve some of the most pressing issues facing society today.

To solve these wicked problems we need to clearly define problem conditions, <b>develop flexible and open workflows</b>, and build the tools that allow us to take advantage of the knowledge we have been developing across disciplines over the last century for data collection, analysis and assimilation.  We must also  <b>develop outreach solutions</b> that help ensure equity within academic and public spheres.

<h2>About Me</h2><hr>

[My publication record](http://simongoring.github.io/cv/Publications.html) focuses on relationships between biotic systems and large scale climatic change at time-scales of centuries to millenia, but my research efforts focus on facilitating broader engagement with the tools and methods neccessary to build the technical toolkits we require to move forward as a society.

Domain specific knowledge is important in managing research questions or looking for solutions, but increasingly, domain knowledge needs to be supported by experience in <b>building research infrastructure, undertaking outreach, managing complex workflows, and planning projects with multiple interconnected components</b>.  My research activities pair deep understanding in the paleogeosciences with project management skills in geoinformatics, as part of the [Neotoma Paleoecological Database](http://neotomadb.org), and within [EarthCube](http://earthcube.org).

<div class="news">
	<h1>News</h1>
	<hr>
	{% for item in site.data.news_items %}
	<details>
	<summary>{{item.content}}</summary>

	{% if item.class == "publication" %}
	<p class="hangingindent" style="padding-left:20px;font-size:0.9em;padding-top:4px">{{item.element}} [<a href="{{item.link}}">link</a>]</p>
	{% endif %}
	{% if item.class == "news" %}
	[<a href="{{item.link}}">link</a>]
	{% endif %}
	</details>
	{% endfor %}
</div>
<br>
<hr>
<small>1: Rittel & Webber [1973](http://www.uctc.net/mwebber/Rittel+Webber+Dilemmas+General_Theory_of_Planning.pdf).  A wicked problem is a complex problem that often has no clear solution, either because there are competing factors, the conditions change, knowledge is incomplete, or because the problem itself is poorly defined.</small>