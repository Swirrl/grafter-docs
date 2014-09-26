---
layout: page
title: Choose an ontology
---

## Choosing an ontology

Only experience can help you become a master in the art of ongologies. [But W3C may be your grandmaster:](http://www.w3.org/standards/semanticweb/ontology)

- [Data Catalog Vocabulary (DCAT)](http://www.w3.org/TR/vocab-dcat/)
- [OWL Web Ontology Language Reference](http://www.w3.org/TR/owl-ref/)
- [RDF Schema 1.1](http://www.w3.org/TR/rdf-schema/)
- [The RDF Data Cube Vocabulary](http://www.w3.org/TR/vocab-data-cube/)
- [vCard Ontology - People and Organizations](http://www.w3.org/TR/vcard-rdf/)
- [PURL](http://purl.org/docs/index.html)
- [Simple Knowledge Organization System (SKOS)](http://www.w3.org/2009/08/skos-reference/skos.html)
- [Vocabulary of Interlinked Datasets](http://vocab.deri.ie/void)
- [Ordnance Survey](http://data.ordnancesurvey.co.uk/ontology/)
- [XML Schema](http://www.w3.org/2001/XMLSchema)

## Grafter's Ontologies

Grafter is here to simplify your RDFization and almost every useful ontology is defined in <code>src/grafter/rdf/ontologies</code>. You can use them easily:

{% highlight clojure %}

user=> (:require [grafter.rdf.ontologies.vcard :refer :all])
user=> vcard:hasUrl

"http://www.w3.org/2006/vcard/ns#hasUrl"

{% endhighlight %}
