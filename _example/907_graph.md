---
layout: page
title: 3. Build the graph
---

# Making graph fragments

## graph-fn
The graph-fn function translates a data table created by the pipeline into a sequence of RDF statements. If multiple tables are provided their resulting statements are combined into a single collection. We declare the header and then define the source tables for the sequence quads:

{% highlight clojure %}

(def glasgow-life-facilities-template
    (graph-fn [[facility-description facility-name monthly-attendance
                        year month address town postcode website facility-type
                        name-slug ref-facility-uri postcode-uri date prefix-date
                        type-name observation-uri]]

                    (graph (base-graph "glasgow-life-facilities")
                          [**triples**])

                    (graph (base-graph "glasgow-life-attendances")
                           [**triples**]))

          dataset))

{% endhighlight %}

## Glasgow life facilities graph
Now we can add our first graph, just by comparing with the final graph:


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
                           [**triples**])))

{% endhighlight %}

## Glasgow life attendances graph
And the second graph:

{% highlight clojure %}

(def glasgow-life-facilities-template
    ((graph-fn [[facility-description facility-name monthly-attendance
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
                       [rdf:a qb:Observation]]))))

{% endhighlight %}
