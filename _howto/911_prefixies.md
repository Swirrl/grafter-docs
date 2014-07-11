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
(ns cmd-line.make-graph
  (:require [clojure.string :as st]
            [grafter.rdf :refer [prefixer s graph graphify]]
            [grafter.rdf.sesame :as ses]
            [grafter.rdf.ontologies.rdf :refer :all]
            [grafter.rdf.ontologies.void :refer :all]
            [grafter.rdf.ontologies.dcterms :refer :all]
            [grafter.rdf.ontologies.vcard :refer :all]
            [grafter.rdf.ontologies.pmd :refer :all]
            [grafter.rdf.ontologies.qb :refer :all]
            [grafter.rdf.ontologies.os :refer :all]
            [grafter.rdf.ontologies.sdmx-measure :refer :all]
            [cmd-line.prefixers :refer :all]
            [cmd-line.pipeline :refer [pipeline]]))
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


## Useful functions
We will also need some functions to transform and adapt data

#### uriify-facility
{% highlight clojure %}
(def uriify-facility {"Museums" (urban "Museum")
                      "Arts" (urban "ArtsCentre")
                      "Community Facility" (urban "CommunityFacility")
                      "Libraries" (urban "Library")
                      "Music" (urban "MusicVenue")
                      "Sport Centres" (urban "SportsCentre")})

cmd-line.prefixers=> (uriify-facility "Libraries")
"http://linked.glasgow.gov.uk/def/urban-assets/Library"
{% endhighlight %}

#### uriify-refFacility
{% highlight clojure %}
(defn uriify-refFacility [type name] 
  (str (urban-id type) "/" name))
    
cmd-line.prefixers=> (uriify-refFacility "foo" "bar")
"http://linked.glasgow.gov.uk/id/urban-assets/foo/bar"
{% endhighlight %}

#### slug-combine
{% highlight clojure %}
(defn slug-combine [& args]
  (apply str (interpose "/" args)))
    
cmd-line.prefixers=> (slug-combine "foo" "bar" "baz")
"foo/bar/baz"
{% endhighlight %}

#### uriify-type
{% highlight clojure %}
(def uriify-type {"Museums" "museums"
                  "Arts" "arts-centres"
                  "Community Facility" "community-facilities"
                  "Libraries" "libraries"
                  "Music" "music-venues"
                  "Sport Centres" "sports-centres"})
    
cmd-line.prefixers=> (uriify-type "Community Facility")
"community-facilities"
{% endhighlight %}

#### date-slug
{% highlight clojure %}
(defn date-slug [date]
  (str (.getYear date) "-" (.getMonthOfYear date) "/"))
{% endhighlight %}

#### slugify-facility
{% highlight clojure %}
(def slugify-facility
  (js-fn "function(name) {
              var lower = name.toLowerCase();
              return lower.replace(/\\ /g, '-');
         }"))

cmd-line.prefixers=> (slugify-facility "Foo bAr")
"foo-bar"
{% endhighlight %}

## Monads

If you don't know about monad yet I recommand [this explication](http://onclojure.com/2009/03/05/a-monad-tutorial-for-clojure-programmers-part-1/) and/or [the documentation](http://clojure.github.io/algo.monads/)

Here are every definition that we will have to use:

{% highlight clojure %}
(with-monad blank-m
  (def rdfstr                    (lift-1 (fn [str] (s str :en))))
  (def replace-comma             (lift-1 (replacer "," ""))  )
  (def trim                      (lift-1 clojure.string/trim))
  (def parse-year                (m-chain [trim replace-comma parse-int]))
  (def parse-attendance          (with-monad identity-m (m-chain [(lift-1 (mapper {"" "0"}))
                                                                     (lift-1 (replacer "," ""))
                                                                     trim
                                                                     parse-int])))
  (def convert-month             (m-chain [trim
                                              (lift-1 clojure.string/lower-case)
                                              (lift-1 {"january" 1 "jan" 1 "1" 1
                                                       "february" 2 "feb" 2 "2" 2
                                                       "march" 3 "mar" 3 "3" 3
                                                       "april" 4 "apr" 4 "4" 4
                                                       "may" 5 "5" 5
                                                       "june" 6 "jun" 6 "6"  6
                                                       "july" 7 "jul" 7 "7"  7
                                                       "august" 8 "aug" 8 "8" 8
                                                       "september" 9 "sep" 9 "sept" 9 "9"  9
                                                       "october" 10 "oct" 10 "10" 10
                                                       "november" 11 "nov" 11 "11" 11
                                                       "december" 12 "dec" 12 "12" 12
                                                       })]))
  (def convert-year              (m-chain [trim parse-int date-time]))
  (def address-line              (m-chain [trim rdfstr]))
  (def city                      (m-chain [trim rdfstr]))
  (def post-code                 (m-chain [trim rdfstr]))
  (def uriify-pcode              (m-chain [trim
                                              (lift-1 (replacer " " ""))
                                              (lift-1 clojure.string/upper-case)
                                              (lift-1 (prefixer "http://data.ordnancesurvey.co.uk/id/postcodeunit/"))]))
  (def url                       (lift-1 #(java.net.URL. %)))
  (def prefix-monthly-attendance (m-chain [(lift-1 date-slug)
                                             (lift-1 (prefixer "http://linked.glasgow.gov.uk/data/glasgow-life-attendances/"))]))
  (def prefix-facility           (prefixer "http://linked.glasgow.gov.uk/data/glasgow-life-attendances/")))
{% endhighlight %}

And now we can understand how everything works:

{% highlight clojure %}
;; rdfstr casts string as an RDF string
cmd-line.prefixers=> (rdfstr "foo")
#<sesame$s$reify__448 foo>

;; replace-comma is quite explicite
cmd-line.prefixers=> (replace-comma "foo,bar")
"foobar"

;; trim removes blanks before and after a string
cmd-line.prefixers=> (trim " foo bar")
"foo bar"

;; parse-year is quite explicite
cmd-line.prefixers=> (parse-year " 2014,")
2014

;; parse-attendance too
cmd-line.prefixers=> (parse-attendance " 0431 ")
431

;; convert-month too
cmd-line.prefixers=> (convert-month " SeptemBer")
9
cmd-line.prefixers=> (convert-month "sep")
9

;; convert-year too
cmd-line.prefixers=> (convert-year 2014)
#<DateTime 2014-01-01T00:00:00.000Z>
  
;; address-line, city and post-code work the same way
cmd-line.prefixers=> (city "Glasgow")
#<sesame$s$reify__448 Glasgow>

;; uriify-pcode, well, urrify post-codes
cmd-line.prefixers=> (uriify-pcode "G0 431 ")
"http://data.ordnancesurvey.co.uk/id/postcodeunit/G0431"

;; url
cmd-line.prefixers=> (url "http://www.swirrl.com")
#<URL http://www.swirrl.com>
  
;; prefix-monthly-attendance 
  FIXME
  
;; prefix-facility
cmd-line.prefixers=> (prefix-facility "foo")
"http://linked.glasgow.gov.uk/data/glasgow-life-attendances/foo" 
{% endhighlight %}

Phew, that's done! Now we can [transform our data...](921_pipeline.html)