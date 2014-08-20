---
layout: page
title: Build a Linked Data template
---

# Graph level

## How to build a Linked Data template

### graph-fn
We can define our templates easily using the graph-fn function. We just need [to choose our ontologies](420_ontology_choice.html) and define, in the graph-fn function, the graph structure we want:

{% highlight clojure %}

((graph-fn [[name street-address city postcode ref-facility-uri]] ;; row binding

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
  dataset)
{% endhighlight %}

![Linked Data template](/assets/310_linked_data_template_3.png)

### Row binding

As a first step: we have defined the binding. Rows are passed to the function one at a time as hash-maps, which can be destructured via Clojure's standard destructuring syntax. Additionally destructuring can be done on row-indicies (when a vector form is supplied) or column names (when a hash-map form is supplied).

{% highlight clojure %}

(let [ds (make-dataset [[1 2 3]] [:a :b :c])]
  ((graph-fn [{:keys [a b c]}]
             (graph "http:///foo.com/"
               [a b c])

             (graph "http:///bar.com/"
               [a b c]))
   ds)))

(let [ds (make-dataset [[1 2 3]])]
  ((graph-fn [{:strs [a b c]}]
              (graph "http:///foo.com/"
               [a b c]))
   ds)))

(let [ds (make-dataset [:a "a" "c"] [[1 2 3]])]
  ((graph-fn [{:keys [a]
               :strs [c]
                a-str "a"}]

              (graph "http:///foo.com/"
                [a a-str c]))
  ds)))

{% endhighlight %}
