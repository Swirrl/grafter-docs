---
layout: page
title: Prefixies
---

# Prefixies

There is no code difficulties here, what could be complicated is: how familiar are you with the data you are working on? How clear the graph you want is in your mind? And are you used to RDF, ontologies and that kind of things. 
For our example we already have the graph we want so we just need to search for every piece of text that we do not already have in our CSV file and define it... 

Just for more readability, here is what we've got and what we want once again:

![Data Screenshot](/assets/911_prefixies_1.png)

{% highlight turtle %}
<http://linked.glasgow.gov.uk/id/urban-assets/museums/riverside-museum> <http://www.w3.org/2000/01/rdf-schema#label> "Riverside Museum"@en ;
	<http://www.w3.org/2006/vcard/nshasUrl> <http://www.glasgowlife.org.uk/museums/riverside/Pages/default.aspx> ;
	a <http://linked.glasgow.gov.uk/def/urban-assets/Museum> , <http://linked.glasgow.gov.uk/def/urban-assets/LeisureFacility> ;
	<http://www.w3.org/2006/vcard/nshasAddress> _:bnode2051 .

_:bnode2051 a <http://www.w3.org/2006/vcard/nsAddress> ;
	<http://www.w3.org/2000/01/rdf-schema#label> "100 Pointhouse Place"@en ;
	<http://www.w3.org/2006/vcard/nsstreet-address> "100 Pointhouse Place"@en ;
	<http://www.w3.org/2006/vcard/nslocality> "Glasgow"@en ;
	<http://www.w3.org/2006/vcard/nscountry-name> "Scotland"@en ;
	<http://www.w3.org/2006/vcard/nspostal-code> "G3 8RS"@en ;
	<http://data.ordnancesurvey.co.uk/ontology/postcode/postcode> <http://data.ordnancesurvey.co.uk/id/postcodeunit/G38RS> .

<http://linked.glasgow.gov.uk/data/glasgow-life-attendances/2013-9/museums/riverside-museum> <http://linked.glasgow.gov.uk/def/refFacility> <http://linked.glasgow.gov.uk/id/urban-assets/museums/riverside-museum> ;
	<http://linked.glasgow.gov.uk/def/numAttendees> "48521"^^<http://www.w3.org/2001/XMLSchema#int> ;
	<http://purl.org/linked-data/cube#dataSet> <http://linked.glasgow.gov.uk/data/glasgow-life-attendances> ;
	<http://data.opendatascotland.org/def/statistical-dimensions/refPeriod> <http://reference.data.gov.uk/id/month/2013-09> ;
	a <http://purl.org/linked-data/cube#Observation> .
{% endhighlight %}

## Dependencies

We are going to need every ontologies defined in Grafter, some functions from grafter.parse grafter.protocols and grafter.js. We also need the [Clojure algo.monads API](http://clojure.github.io/algo.monads/) and the clojure string API:

{% highlight clojure %}
(ns cmd-line.prefixers
  (:require [grafter.csv :refer [fuse derive-column parse-csv mapc swap drop-rows _]]
           [grafter.rdf.protocols :as pr]
           [clojure.string :as st]
           [grafter.rdf :refer [prefixer s]]
           [grafter.rdf.ontologies.rdf :refer :all]
           [grafter.rdf.ontologies.void :refer :all]
           [grafter.rdf.ontologies.dcterms :refer :all]
           [grafter.rdf.ontologies.vcard :refer :all]
           [grafter.rdf.ontologies.pmd :refer :all]
           [grafter.rdf.ontologies.qb :refer :all]
           [grafter.rdf.ontologies.os :refer :all]
           [grafter.rdf.ontologies.sdmx-measure :refer :all]
           [grafter.parse :refer [lift-1 blank-m replacer mapper parse-int date-time]]
           [grafter.js :refer [js-fn]]
           [clojure.algo.monads :refer [m-chain m-bind m-result with-monad identity-m]]))
{% endhighlight %}

## Base prefixies
Base prefixies are all defined on the same pattern, using the Grafter's prefixer function:  

{% highlight clojure %}
(def base-uri (prefixer "http://linked.glasgow.gov.uk"))

cmd-line.prefixers=> (base-uri "/blah")
"http://linked.glasgow.gov.uk/blah"
{% endhighlight %}

So you just have to check the final graph and define every bases of uri:

{% highlight clojure %}
(def base-uri (prefixer "http://linked.glasgow.gov.uk"))
(def base-graph (prefixer (base-uri "/graph/")))

(def glasgow (prefixer "http://linked.glasgow.gov.uk/def/"))
(def urban (prefixer "http://linked.glasgow.gov.uk/def/urban-assets/"))
(def urban-id (prefixer "http://linked.glasgow.gov.uk/id/urban-assets/"))
(def ont-graph "http://linked.glasgow.gov.uk/graph/vocab/urban-assets/ontology")

(def attendance (prefixer "http://linked.glasgow.gov.uk/data/facility_attendance")) 
(def urban:ontology (urban "ontology"))
(def sd (prefixer "http://data.opendatascotland.org/def/statistical-dimensions/"))
{% endhighlight %}





