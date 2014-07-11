---
layout: page
title: Exemple Glasgow life facilities
---

# Example Glasgow life facilities
This exemple aims, with a concrete case, both to present you how you can use Grafter for your RDFization projects and to help you understand what is happening in the engine.

## Data presentation
We are going to use the dataset "test-data/glasgow-life-facilities.csv".
Data looks like this:
![Data Screenshot](/assets/905_general_1.png)

## What do we want?
Our goal is to get a Turtle file with triples like this (example for the first raw - Riverside Museum):

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

And basically, we would like to get it from the command line with something like:

{% highlight shell %}
$ lein run my-csv-file my-output-graph
{% endhighlight %}

## Global process
- The first part of the process is going to be the [definition of every prefixies](911_prefixies.html) we are going to need for our triples
- Then we will [work and transform the data](921_pipeline.html) we have to make them usable
- After that we will be able to [create the graphs we want](931_graph.html)
- There will therefore be only one little step: [filter and export](941_filter_import.html)

## Leiningen project
As a step 0, let's configure a Leiningen project:

{% highlight shell %}
$ lein new cmd-line
$ cd cmd-line
{% endhighlight %}

TOFIX: and in project.clj:

{% highlight clojure %}
(defproject cmd-line "0.1.0-SNAPSHOT"
  :description "FIXME: write description"
  :url "http://example.com/FIXME"
  :license {:name "Eclipse Public License"
            :url "http://www.eclipse.org/legal/epl-v10.html"}
  :dependencies [[org.clojure/clojure "1.6.0"]
                 [grafter "0.1.0"]]
  :main cmd-line.core
  :plugins [[s3-wagon-private "1.1.2"]])
{% endhighlight %}

Now let's graft some CSV!