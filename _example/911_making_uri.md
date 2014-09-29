---
layout: page
title: Building URIs
---

## Building URIs

Good URIs are an essential component of RDF data sets, and a common operation is building URIs from a common prefix. You can use the prefixer function to build URIs with a common base.

{% highlight clojure %}

(def base-uri (prefixer "http://linked.glasgow.gov.uk"))

user=> (base-uri "/blah")
"http://linked.glasgow.gov.uk/blah"

{% endhighlight %}

So you just have to check the final graph and define every URI base:

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

(def uriify-facility {"Museums" (urban "Museum")
                      "Arts" (urban "ArtsCentre")
                      "Community Facility" (urban "CommunityFacility")
                      "Libraries" (urban "Library")
                      "Music" (urban "MusicVenue")
                      "Sport Centres" (urban "SportsCentre")})

(def prefix-facility (prefixer "http://linked.glasgow.gov.uk/data/glasgow-life-attendances/"))

(defn uriify-ref-facility [type name]
  (str (urban-id type) "/" name))

{% endhighlight %}

{% highlight clojure %}

user=> (uriify-facility "Libraries")
"http://linked.glasgow.gov.uk/def/urban-assets/Library"

user=> (uriify-ref-facility "foo" "bar")
"http://linked.glasgow.gov.uk/id/urban-assets/foo/bar"

{% endhighlight %}


## Slugify

If you have data containing spaces and mixed-case characters which you want to convert into a URI fragment, you may want to transform it into a [slug](http://patterns.dataincubator.org/book/url-slug.html). The slugify function used in this tutorial removes any surrounding whitespace, converts the string to lower-case and replaces any internal spaces into a '-'. Its definition is shown below:

{% highlight clojure %}

(defn slugify [string]
  (-> string
      st/trim
      (st/lower-case)
      (st/replace " " "-")))

(defn slug-combine [& args]
  (apply str (interpose "/" args)))

(defn date-slug [date]
  (str (.getYear date) "-" (.getMonthOfYear date) "/"))

{% endhighlight %}


{% highlight clojure %}

user=> (slug-combine "foo" "bar" "baz")
"foo/bar/baz"

user=> (slugify " Foo bAr")
"foo-bar"

{% endhighlight %}

[Now you can see how to filter and export](941_filter_import.html)
