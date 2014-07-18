---
layout: page
title: Build a Linked Data template
---

# Graph level

## How to build a Linked Data template

### Graphify

![Linked Data template](/assets/310_linked_data_template_1.png)

TOFIX
{% highlight clojure %}
(defn make-life-facilities [csv-path]
  (let [processed-rows (pipeline csv-path)]

         ((graphify [facility-uri name attendance date street-address city postcode website facility-type name-uri ref-facility-uri
                     postcode-uri observation-uri].....................
{% endhighlight %}

![Linked Data template](/assets/310_linked_data_template_2.png)

### Graph

TOFIX
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
                                              [os:postcode postcode-uri]]]]).....................
{% endhighlight %}

![Linked Data template](/assets/310_linked_data_template_3.png)
