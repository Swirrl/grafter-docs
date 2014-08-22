---
layout: page
title: Leiningen project
---

# Leiningen project
As a step 0, let's configure a Leiningen project:

{% highlight shell %}

$ lein new glasgow-life-facilities
$ cd glasgow-life-facilities

{% endhighlight %}

And in project.clj:

{% highlight clojure %}

(defproject glasgow-life-facilities "0.1.0-SNAPSHOT"
  :description "Example of RDFization using Grafter 0.2.0 on the csv file
  Glasgow Life Facilities"
  :url "http://example.com/FIXME"
  :license {:name "Eclipse Public License"
            :url "http://www.eclipse.org/legal/epl-v10.html"}
  :dependencies [[org.clojure/clojure "1.6.0"]
                 [grafter "0.2.0"]]
  :main glasgow-life-facilities.core
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

In src/glasgow-life-facilities/prefixers.clj


{% highlight clojure %}

(ns glasgow-life-facilities.prefixers
  (:require [grafter.rdf.protocols :as pr]
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
            [clojure.algo.monads :refer [m-chain m-bind m-result with-monad identity-m]]))

{% endhighlight %}

### Prefixers code

{% highlight clojure %}

;;; URI

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


;;; Slugify

(defn slugify [string]
  (-> string
      st/trim
      (st/lower-case)
      (st/replace " " "-")))

(defn slug-combine [& args]
  (apply str (interpose "/" args)))

(defn date-slug [date]
  (str (.getYear date) "-" (.getMonthOfYear date) "/"))


;;; Data cleaning

(def clean-type {"Museums" "museums"
                  "Arts" "arts-centres"
                  "Community Facility" "community-facilities"
                  "Libraries" "libraries"
                  "Music" "music-venues"
                  "Sport Centres" "sports-centres"})

(with-monad blank-m
  (def rdfstr                    (lift-1 (fn [str] (s str :en))))
  (def replace-comma             (lift-1 (replacer "," ""))  )
  (def trim                      (lift-1 st/trim))
  (def parse-year                (m-chain [trim replace-comma parse-int]))
  (def parse-attendance          (with-monad identity-m (m-chain [(lift-1 (mapper {"" "0"}))
                                                                     (lift-1 (replacer "," ""))
                                                                     trim
                                                                     parse-int])))
  (def convert-month             (m-chain [trim
                                              (lift-1 st/lower-case)
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
                                              (lift-1 st/upper-case)
                                              (lift-1 (prefixer "http://data.ordnancesurvey.co.uk/id/postcodeunit/"))]))
  (def url                       (lift-1 #(java.net.URL. %)))
  (def prefix-monthly-attendance (m-chain [(lift-1 date-slug)
                                             (lift-1 (prefixer "http://linked.glasgow.gov.uk/data/glasgow-life-attendances/"))])))

{% endhighlight %}

## Pipeline

### Pipeline dependencies

In src/glasgow-life-facilities/pipeline.clj

{% highlight clojure %}

(ns glasgow-life-facilities.pipeline
  (:require [grafter.tabular :refer [column-names derive-column mapc swap drop-rows _]]
            [grafter.tabular.common :refer [open-all-datasets make-dataset move-first-row-to-header]]
            [grafter.parse :refer [date-time]]
            [glasgow-life-facilities.prefixers :refer :all]))

{% endhighlight %}

### Pipeline code

{% highlight clojure %}

(defn pipeline [dataset]
  (-> dataset
      (make-dataset ["facility-description" "facility-name" "monthly-attendance" "month" "year" "address" "town" "postcode" "website"])
      (drop-rows 1)
      (derive-column "facility-type" ["facility-description"] clean-type)
      (derive-column "name-slug" ["facility-name"] slugify)
      (mapc {"facility-description" uriify-facility
             "monthly-attendance" parse-attendance
             "month" convert-month
             "year" parse-year
             "address" address-line
             "town" city
             "postcode" post-code
             "website" url})
      (derive-column "ref-facility-uri" ["facility-type" "name-slug"] uriify-ref-facility)
      (derive-column "postcode-uri" ["postcode"] uriify-pcode)
      (swap "month" "year")
      (derive-column "date" ["year" "month"] date-time)
      (derive-column "prefix-date" ["date"] prefix-monthly-attendance)
      (derive-column "type-name" ["facility-type" "name-slug"] slug-combine)
      (derive-column "observation-uri" ["prefix-date" "type-name"] str)))

{% endhighlight %}

## Make graph

### Make graph dependencies

In src/glasgow-life-facilities/make_graph.clj

{% highlight clojure %}

(ns glasgow-life-facilities.make-graph
  (:require [grafter.rdf :refer [graph graph-fn]]
            [grafter.rdf.ontologies.rdf :refer :all]
            [grafter.rdf.ontologies.void :refer :all]
            [grafter.rdf.ontologies.dcterms :refer :all]
            [grafter.rdf.ontologies.vcard :refer :all]
            [grafter.rdf.ontologies.pmd :refer :all]
            [grafter.rdf.ontologies.qb :refer :all]
            [grafter.rdf.ontologies.os :refer :all]
            [grafter.rdf.ontologies.sdmx-measure :refer :all]
            [glasgow-life-facilities.prefixers :refer :all]))

{% endhighlight %}

### Make graph code

{% highlight clojure %}

(def glasgow-life-facilities-template
  (graph-fn [[facility-description facility-name monthly-attendance
              year month address town postcode website facility-type
              name-slug ref-facility-uri postcode-uri date prefix-date
              type-name observation-uri]]

            (graph (base-graph "glasgow-life-facilities")
                   [ref-facility-uri
                    [rdfs:label (rdfstr facility-name)]
                    [vcard:hasUrl website]
                    [rdf:a (urban "Museum")]
                    [rdf:a (urban "LeisureFacility")]
                    [vcard:hasAddress [[rdf:a vcard:Address]
                                       [rdfs:label address]
                                       [vcard:street-address address]
                                       [vcard:locality town]
                                       [vcard:country-name (rdfstr "Scotland")]
                                       [vcard:postal-code postcode]
                                       [os:postcode postcode-uri]]]])

            (graph (base-graph "glasgow-life-attendances")
                   [observation-uri
                    [(glasgow "refFacility") ref-facility-uri]
                    [(glasgow "numAttendees") monthly-attendance]
                    [qb:dataSet "http://linked.glasgow.gov.uk/data/glasgow-life-attendances"]
                    [(sd "refPeriod") "http://reference.data.gov.uk/id/month/2013-09"]
                    [rdf:a qb:Observation]])))

{% endhighlight %}


## Filter

### Filter dependencies

In src/glasgow-life-facilities/filter.clj

{% highlight clojure %}

(ns glasgow-life-facilities.filter
  (:require [grafter.rdf.protocols :as pr]
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

In src/glasgow-life-facilities/core.clj

{% highlight clojure %}

(ns glasgow-life-facilities.core
  (:require [grafter.tabular :refer :all]
            [grafter.rdf.protocols :as pr]
            [grafter.rdf.sesame :as ses]
            [grafter.rdf.validation :refer [has-blank? validate-triples]]
            [glasgow-life-facilities.filter :refer [filter-triples]]
            [glasgow-life-facilities.make-graph :refer [glasgow-life-facilities-template]]
            [glasgow-life-facilities.pipeline :refer [pipeline]]))

{% endhighlight %}

### import

{% highlight clojure %}

(defonce my-repo (-> "./tmp/grafter-sesame-store-2" ses/native-store ses/repo))

(defn import-life-facilities
  [quads-seq destination]
  (let [now (java.util.Date.)
        quads (->> quads-seq
                   filter-triples
                   (validate-triples (complement has-blank?)))]

    (pr/add (ses/rdf-serializer destination) quads)))

{% endhighlight %}

### -main

Remember in our project.clj there were this line:

{% highlight clojure %}
:main glasgow-life-facilities.core
{% endhighlight %}

We have to define a -main function, this function will take input and output as argument:

{% highlight clojure %}

(defn -main [path output]
  (-> (open-all-datasets path)
      first
      pipeline
      glasgow-life-facilities-template
      (import-life-facilities output))
  (println path "has been grafted using Grafter 0.2.0!"))

{% endhighlight %}

## Test

{% highlight shell %}

$ lein run ./data/glasgow-life-facilities.csv glasgow-life-facilities.ttl
./data/glasgow-life-facilities.csv has been grafted using Grafter 0.2.0!

{% endhighlight %}

![Result!](/assets/951_command_line_1.png)

## Conclusion

What's important in this example is the global process and the global philosophy, but must of all, [the pipeline function](906_pipeline.html) is the key!
