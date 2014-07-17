---
layout: page
title: Graph
---

# Graph

We [now have every thing we needed](921_pipeline.html) to make our graphs!


## Graphify
The idea here is tu use the Grafter function 'graphify' on each row processed on the pipeline function. We map each row -so we can call directly each column on our triples, and, then, we write sequence quads:

{% highlight clojure %}

(defn make-life-facilities [csv-path]
  (let [processed-rows (pipeline csv-path)]

         ((graphify [facility-uri name attendance date street-address city postcode website facility-type name-uri ref-facility-uri
                     postcode-uri observation-uri]

                    (graph (base-graph "glasgow-life-facilities")
                          [**triples**])

                    (graph (base-graph "glasgow-life-attendances")
                           [**triples**]))

          processed-rows)))
{% endhighlight %}

## Glasgow life facilities graph
Now we can add our first graph, just by comparing with the final graph:


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
                                              [os:postcode postcode-uri]]])

                    (graph (base-graph "glasgow-life-attendances")
                           [**triples**]))

          processed-rows)))
{% endhighlight %}

Just to represent yourself what it can be, after integration of the graph in [PublishMyData](http://www.swirrl.com/publishmydata), our glasgow life facilities graph will look like this:

![pmd screenshot](/assets/931_graph_1.png)

![pmd screenshot](/assets/931_graph_2.png)

## Glasgow life attendances graph
And the second graph:

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
                                              [os:postcode postcode-uri]]])

                    (graph (base-graph "glasgow-life-attendances")
                           [observation-uri
                            [(glasgow "refFacility") ref-facility-uri]
                            [(glasgow "numAttendees") attendance]
                            [qb:dataSet "http://linked.glasgow.gov.uk/data/glasgow-life-attendances"]
                            [(sd "refPeriod") "http://reference.data.gov.uk/id/month/2013-09"]
                            [rdf:a qb:Observation]]))

          processed-rows)))
{% endhighlight %}

![pmd screenshot](/assets/931_graph_3.png)


## Conclusion

Almost there! We now just need to [filter our triples, export them](941_filter_import.html) and see how to ["lein" everything!](951_command_line.html)
