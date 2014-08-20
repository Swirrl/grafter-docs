---
layout: page
title: Make graph fragments
---

# Make graph fragments

## Graph-fn
The idea here is tu use the Grafter function 'graph-fn' on each row processed on the pipeline function. We bind the header - so we can call directly each column on our triples, and, then, we write sequence quads:

{% highlight clojure %}

defn make-life-facilities [path]
  (let [dataset (pipeline path)]

    ((graph-fn [[facility-description facility-name monthly-attendance
                        year month address town postcode website facility-type
                        name-slug ref-facility-uri postcode-uri date prefix-date
                        type-name observation-uri]]

                    (graph (base-graph "glasgow-life-facilities")
                          [**triples**])

                    (graph (base-graph "glasgow-life-attendances")
                           [**triples**]))

          dataset)))

{% endhighlight %}

## Glasgow life facilities graph
Now we can add our first graph, just by comparing with the final graph:


{% highlight clojure %}

(defn make-life-facilities [path]
  (let [dataset (pipeline path)]

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
                           [**triples**]))

          dataset)))

{% endhighlight %}

Just to represent yourself what it can be, after integration of the graph in [PublishMyData](http://www.swirrl.com/publishmydata), our glasgow life facilities graph will look like this:

![pmd screenshot](/assets/931_graph_1.png)

![pmd screenshot](/assets/931_graph_2.png)

## Glasgow life attendances graph
And the second graph:

{% highlight clojure %}

(defn make-life-facilities [path]
  (let [dataset (pipeline path)]

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
                       [rdf:a qb:Observation]]))

     dataset)))

{% endhighlight %}

![pmd screenshot](/assets/931_graph_3.png)


## Conclusion

The Pipeline function and this one are really the most important, but the work is not over! There are still some [data cleaning](908_cleaning.html), some [URI making](911_making_uri.html), some
[filtering and export](941_filter_import.html) and, finally, see how to ["lein" everything!](951_command_line.html)
