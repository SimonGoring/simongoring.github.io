---
title: Ontology in Earth Science
concept: Thoughts about ontology systems and their applicability in the Earth Sciences
layout: page
---


At the [linked.earth](http://www.organicdatacuration.org/linkedearth/index.php/Main_Page) meeting in Boulder, CO (June 22 - 23) we really started digging into [ontologies](https://en.wikipedia.org/wiki/Ontology_(information_science)).  In building the DOI system and landing pages for Neotoma I've been thinking about ways to build out our system so that more information can be contained in the Landing Page documents, extending the ability of web technologies to harvest & gather information about the scientific content embedded in the records.  This has led me to several key resources for building web-readable content to supplement landing pages.

## `FOAF`

`foaf` is a pretty standard ontology.  It is named after the acronym "Friend of a Friend" and is used to describe people, their relationships and their activities.  [The foaf specification](http://xmlns.com/foaf/spec/) is very extensive, and, frankly, I've found the documentation somewhat opaque.  That said, I'm still fairly new at working my way through these documents.

The key use for `foaf` is describing our author data in Neotoma.  Using either JSON or RDF/XML it's possible to directly translate most author and institutional data to some form of RDF.  I wrote out a code snippet that I use as header in my webpage (it's written as a `<script>` element, so you can't see it) that describes who I am, what projects I belong to (Neotoma & EarthCube).  This uses RDF/XML, but my feeling is that I'll probably rewrite this as JSON-LD, especially since the standards are so nicely written! (FYI - There's a great [blog post about the standards here](http://manu.sporny.org/2014/json-ld-origins-2/)).

```XML
<rdf:RDF
      xmlns:rdf="http://www.w3.org/1999/02/22-rdf-syntax-ns#"
      xmlns:rdfs="http://www.w3.org/2000/01/rdf-schema#"
      xmlns:foaf="http://xmlns.com/foaf/0.1/"
      xmlns:admin="http://webns.net/mvcb/">
	<foaf:Person rdf:ID="me">
		<foaf:name xml:lang="en">Simon J. Goring</foaf:name>
		<foaf:givenname>Simon</foaf:givenname>
		<foaf:family_name>Goring</foaf:family_name>
		<foaf:homepage rdf:resource="http://simongoring.github.io"/>
		<foaf:publications rdf:resource="http://simongoring.github.io/cv/Publications.html"/>
		<foaf:schoolHomepage rdf:resource="http://www.wisc.edu"/>
		<foaf:mbox_sha1sum>4f9fb359d3396b0eda825202f843508eaeb26f79</foaf:mbox_sha1sum>
		<foaf:account>
      		<foaf:OnlineAccount>
        		<foaf:accountServiceHomepage rdf:resource="http://www.orcid.org"/>
        <foaf:accountName>0000-0002-2700-4605</foaf:accountName>
        <foaf:accountProfilePage rdf:resource="http://www.orcid.org/0000-0002-2700-4605#person"/>
      </foaf:OnlineAccount>
    </foaf:account>
    <foaf:account>
  		<foaf:OnlineAccount>
    		<foaf:accountServiceHomepage rdf:resource="http://www.twitter.com/"/>
    		<foaf:accountName>sjGoring</foaf:accountName>
    		<foaf:accountProfilePage rdf:resource="http://www.twitter.com/sjGoring"/>
  		</foaf:OnlineAccount>
	</foaf:account>
    <foaf:currentProject>
      <foaf:Project>
	      <foaf:organization>Neotoma Paleoecological Database</foaf:organization>
	      <foaf:homepage rdf:resource="http://neotomadb.org"/>
      </foaf:Project>
     </foaf:currentProject>
     <foaf:currentProject>
      <foaf:Project>
      	<foaf:organization>EarthLife Consortium</foaf:organization>
        <foaf:homepage rdf:resource="http://earthlifeconsortium.org" />
      </foaf:Project>
    </foaf:currentProject>
  </foaf:Person>
</rdf:RDF>
```

## `PROV`

Prov is an ontology used to describe the provenance of objects and activities.  It can focus on the agent of change in the system, the object that is changed across the chain of provenance, or the activities, carried out by the agents on a set of objects.  The [PROV](https://www.w3.org/TR/2013/NOTE-prov-primer-20130430/) data model is of particular interest.  Part of our motivation for beginning to mint DOIs for [Neotoma](http://neotomadb.org) -- described more fully [here]() -- was so that we could provide them as currency for the exchange of data between related databases.

`PROV` would allow us to link to resources like [OpenCoreData](http://opencoredata.org/) or [LacCore](http://lrc.geo.umn.edu/laccore/), where the lake sediment for the lacustrine cores might be stored.  We could establish a chain of provenance for materials and measurements from the field to publication.

## `owl-time`

The [time ontology in OWL](https://www.w3.org/TR/owl-time/) is more complex and we're working through it slowly.  In particular, pollen samples are obtained from sediment within a core, a physical interval with (initially) unknown temporal bounds.  Even when we build a chronology, most pollen samples are not directly dated, and dates for individual depth intervals are known with non-normal uncertainty bounds.

<figure>
<img src="/images/calib.gif" alt="Radiocarbon calibration curve">
<figcaption><i>A schema for calibration of <sup>14</sup>C dates from the <a href="https://c14.arch.ox.ac.uk/embed.php?File=calibration.html">University of Oxford's radiocarbon Web-info</a>, showing the non-normal distribution of calibrated radiocarbon dates.</i></figcaption></figure>

As far as we can see, it is a challenge to have adjacent physical samples with overlapping age bounds, but a constraint that ages are monotonically increasing with depth (in theory).  A full implementation would have to pair descriptions of `thing`s, time, provenance, some description of the methods for chronology generation.  In essence, all of the work we've done with Neotoma to fully (and successfully) model paleoecological data needs to be translated into a web-accessible ontology.

This is not simple work, and needs some time to sort out.