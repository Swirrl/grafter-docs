---
layout: page
title: Leiningen project
---

# Leiningen project
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

And here is what our src/cmd-line/ file should look like:

.

├── core.clj

├── filter.clj

├── make_graph.clj

├── pipeline.clj

└── prefixers.clj


## Prefixers

### Prefixers dependencies

In src/cmd-line/prefixers.clj

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

### Prefixers code

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

(defn uriify-refFacility [type name]
  (str (urban-id type) "/" name))

(defn slug-combine [& args]
  (apply str (interpose "/" args)))

(def uriify-type {"Museums" "museums"
                  "Arts" "arts-centres"
                  "Community Facility" "community-facilities"
                  "Libraries" "libraries"
                  "Music" "music-venues"
                  "Sport Centres" "sports-centres"})

(defn date-slug [date]
  (str (.getYear date) "-" (.getMonthOfYear date) "/"))

(def slugify-facility
  (js-fn "function(name) {
              var lower = name.toLowerCase();
              return lower.replace(/\\ /g, '-');
         }"))

(def prefix-facility (prefixer "http://linked.glasgow.gov.uk/data/glasgow-life-attendances/"))

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
                                             (lift-1 (prefixer "http://linked.glasgow.gov.uk/data/glasgow-life-attendances/"))])))


{% endhighlight %}


## Pipeline

### Pipeline dependencies

In src/cmd-line/pipeline.clj

The pipeline function is going to require some Grafter functions:

- parse-csv: simply uses the [Clojure.java.io API reader function](http://clojure.github.io/clojure/clojure.java.io-api.html) to parse our csv file
- drop-rows: drops the first n rows from the CSV
- swap: swaps two columns
- derive-column: adds a new column to the end of the row which is derived from already existing columns
- mapc: takes an array of functions and maps each to the equivalent column position for every row
- fuse: merges columns
- _: identity
- date-time: uses the [clj-time](https://github.com/clj-time/clj-time)'s date-time

And also every prefixies we have defined [at the last step](911_prefixies.html)


{% highlight clojure %}
(ns cmd-line.pipeline
  (:require [grafter.csv :refer [fuse derive-column parse-csv mapc swap drop-rows _]]
            [grafter.parse :refer [date-time]]
            [cmd-line.prefixers :refer :all]))
{% endhighlight %}

### Pipeline code

{% highlight clojure %}
(defn pipeline [path-csv]
 (-> (parse-csv path-csv)
     (drop-rows 1)
     (derive-column uriify-type 0)
     (derive-column slugify-facility 1)
     (mapc [uriify-facility trim parse-attendance convert-month parse-year address-line city post-code url _ _])
     (derive-column uriify-refFacility 9 10)
     (derive-column uriify-pcode 7)
     (swap {3 4})
     (fuse date-time 3 4)
     (derive-column prefix-monthly-attendance 3)
     (derive-column slug-combine 8 9)
     (fuse str 12 13)))

{% endhighlight %}

## Make graph

### Make graph dependencies

In src/cmd-line/make_graph.clj

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

### Make graph code

{% highlight clojure %}

(defn make-life-facilities [csv-path]
  (let [processed-rows (pipeline csv-path)]

         ((graphify [facility-uri name attendance date street-address city postcode website facility-type name-uri ref-facility-uri
                     postcode-uri observation-uri]

                    (graph (base-graph "glasgow-life-facilities")
                          [ref-facility-uri
                            [rdfs:label (rdfstr name)]
                            [vcard:hasUrl website]
                            [rdf:a (urban "Museum")]
                            [rdf:a (urban "LeisureFacility")]
                            [vcard:hasAddress [[rdf:a vcard:Address]
                                              [rdfs:label street-address]
                                              [vcard:street-address street-address]
                                              [vcard:locality city]
                                              [vcard:country-name (rdfstr "Scotland")]
                                              [vcard:postal-code postcode]
                                              [os:postcode postcode-uri]]]])

                    (graph (base-graph "glasgow-life-attendances")
                           [observation-uri
                            [(glasgow "refFacility") ref-facility-uri]
                            [(glasgow "numAttendees") attendance]
                            [qb:dataSet "http://linked.glasgow.gov.uk/data/glasgow-life-attendances"]
                            [(sd "refPeriod") "http://reference.data.gov.uk/id/month/2013-09"]
                            [rdf:a qb:Observation]]))

          processed-rows)))

{% endhighlight %}


## Filter

### Filter dependencies
In src/cmd-line/filter.clj

{% highlight clojure %}
(ns cmd-line.filter
  (:require [grafter.rdf.protocols :as pr]
            [grafter.rdf :refer [prefixer s graph graphify]]
            [grafter.rdf.validation :refer [blank?]]
            [grafter.rdf.ontologies.vcard :refer :all]
            [grafter.rdf.ontologies.os :refer :all]))
{% endhighlight %}

### Filter code

{% highlight clojure %}

(defn filter-triples [triples]
  (filter #(not (and (#{vcard:postal-code os:postcode vcard:hasUrl} (pr/predicate %1))
                     (blank? (pr/object %1)))) triples))

{% endhighlight %}

## Core

### Core dependencies
In src/cmd-line/core.clj

{% highlight clojure %}
(ns cmd-line.core
  (:use [cmd-line.make-graph]
        [cmd-line.prefixers]
        [cmd-line.filter]
        [cmd-line.pipeline])
  (:require [cmd-line.filter :refer [filter-triples]]
            [cmd-line.make-graph :refer [make-life-facilities]]
            [grafter.rdf.protocols :as pr]
            [grafter.rdf.sesame :as ses]
            [grafter.rdf.validation :refer [validate-triples has-blank?]]))
{% endhighlight %}

### import

{% highlight clojure %}

(defonce my-repo (-> "./tmp/grafter-sesame-store2" ses/native-store ses/repo))

(defn import-life-facilities [quads-seq destination]
  (let [now (java.util.Date.)
        quads (->> quads-seq
                   filter-triples
                   (validate-triples (complement has-blank?)))]

    (pr/add (ses/rdf-serializer destination) quads)))

{% endhighlight %}

### -main

Remember in our project.clj there were this line:

{% highlight clojure %}
:main cmd-line.core
{% endhighlight %}

We have to define a -main function, this function will take input and output as argument:

{% highlight clojure %}
(defn -main [my-csv output]
  (println "About to graft " my-csv)

  (import-life-facilities (make-life-facilities my-csv) output)

  (println my-csv "has been grafted: " output))
{% endhighlight %}

## Test

{% highlight shell %}

$ lein run ./test-data/glasgow-life-facilities.csv glasgow-life-facilities.ttl

About to graft  ./test-data/glasgow-life-facilities.csv
./test-data/glasgow-life-facilities.csv has been grafted:  glasgow-life-facilities.ttl

{% endhighlight %}

![Result!](/assets/951_command_line_1.png)

## Conclusion

What's important in this example is the global process and the global philosophy, but must of all, [the pipeline function](921_pipeline.html) is the key!
