---
layout: page
title: Making URI
---

# Making URI

There is no code difficulties here, what could be complicated is: how familiar are you with the data you are working on? How clear the graph you want is in your mind? And are you used to RDF, ontologies and that kind of things.
For our example we already have the graph we want so we just need to search for every piece of text that we do not already have in our CSV file and define it...

Just for more readability, here is what we've got:

{% highlight CSV %}

"Museums", "Riverside Museum", 48521, September, 2013, "100 Pointhouse Place", "Glasgow", "G3 8RS", "http://www.glasgowlife.org.uk/museums/riverside/Pages/default.aspx"

{% endhighlight %}

and what we want:

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


## Base prefixes
Base prefixes are all defined on the same pattern, using the Grafter's prefixer function:

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


[Now we can transform our data! That's the nice part of this tutorial!](921_pipeline.html)
