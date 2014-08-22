---
layout: page
title: The DSL
---

# The Grafter DSL

At the heart of Grafter is a Domain Specific Langauge (DSL) which
allows the specification of transformation pipelines that convert
tabular data into linked data formats and the creation of graphs.

The DSL is designed to support a good separation of concerns, and
acknowledges the reality that even within a single dataset many people
with different skills and responsibilities may be involved.

## Grafter Tabular DSL

The [Grafter Tabular DSL](http://api.grafter.org/0.2/grafter.tabular.html){:target="_blank"} allows you to create transformation pipelines to transform the data, preserving source data:

{% highlight clojure %}

(defn pipeline [dataset]
  (-> dataset
      (make-dataset ["facility-description" "facility-name" "monthly-attendance" "month" "year" "address" "town" "postcode" "website"])
      (drop-rows 1)
      (derive-column "facility-type" ["facility-description"] uriify-type)
      (derive-column "name-slug" ["facility-name"] slugify)
      (mapc {"facility-description" uriify-facility
             "monthly-attendance" parse-attendance
             "month" convert-month
             "year" parse-year
             "address" address-line
             "town" city
             "postcode" post-code
             "website" url})
      (derive-column "ref-facility-uri" ["facility-type" "name-slug"] uriify-refFacility)
      (derive-column "postcode-uri" ["postcode"] uriify-pcode)
      (swap "month" "year")
      (derive-column "date" ["year" "month"] date-time)
      (derive-column "prefix-date" ["date"] prefix-monthly-attendance)
      (derive-column "type-name" ["facility-type" "name-slug"] slug-combine)
      (derive-column "observation-uri" ["prefix-date" "type-name"] str)))

{% endhighlight %}

## Grafter Graph DSL

The [Grafter Graph DSL](http://api.grafter.org/0.2/grafter.rdf.html){:target="_blank"} allows you to build linked data templates:

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

## Composition

Those two parts of the core DSL can then be composed easily and the whole process of RDFization will look like this:

{% highlight clojure %}

(-> (open-all-datasets path)
    first
    pipeline
    glasgow-life-facilities-template
    (import-life-facilities output))

{% endhighlight %}
