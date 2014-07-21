---
layout: page
title: Build a Linked Data template
---

# Graph level

## How to build a Linked Data template

### Graphify

As a first step: we define explicits names:

{% highlight clojure %}
(defn make-linked-data-template
  (let [processed-rows]

         ((graphify [name street-address city postcode ref-facility-uri]
                    (...))
                    processed-rows)))
{% endhighlight %}

![Linked Data template](/assets/310_linked_data_template_1.png)

and we get an easy access to data in our graph fragments definition.

![Linked Data template](/assets/310_linked_data_template_2.png)

### Graph
And then we can our template easily. We just need [to choose our ontologies](420_ontology_choice.html) and define the graph structure we want:

{% highlight clojure %}
(defn make-linked-data-template
  (let [processed-rows]

         ((graphify [name street-address city postcode ref-facility-uri]

                    (graph (base-graph "glasgow-life-facilities")
                          [ref-facility-uri
                            [rdfs:label (rdfstr name)]
                            [vcard:hasAddress [[rdf:a vcard:Address]
                                              [rdfs:label street-address]
                                              [vcard:street-address street-address]
                                              [vcard:locality city]
                                              [vcard:country-name (rdfstr "Scotland")]
                                              [vcard:postal-code postcode]]]])
                    (graph...))
                    processed-rows)))
{% endhighlight %}

![Linked Data template](/assets/310_linked_data_template_3.png)
