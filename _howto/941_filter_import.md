---
layout: page
title: Filter and import
---
# Filter and import

It's now time to import those quads into a real graph on a Turtle file. BUT if we try now it won't work because of some blanks in our dataset (some missing url basically). When Grafter writes the triples, it validates them and Grafter does not accept missing data. Thus we have to filter those triples so that, if there is a missing data, the whole triple is not written.

## Filter

### Dependencies
In a src/cmd-line/filter.clj file.

{% highlight clojure %}
(ns cmd-line.filter
  (:require [grafter.rdf.protocols :as pr]
            [grafter.rdf :refer [prefixer s graph graphify]]
            [grafter.rdf.validation :refer [blank?]]
            [grafter.rdf.ontologies.vcard :refer :all]
            [grafter.rdf.ontologies.os :refer :all]))
{% endhighlight %}

### filter-triples
When checking our dataset we can see that there are missing data in the columns 
- Postcode
- Website

{% highlight clojure %}
(defn filter-triples [triples]
  (filter #(not (and (#{vcard:postal-code os:postcode vcard:hasUrl} (pr/predicate %1))
                     (blank? (pr/object %1)))) triples))
{% endhighlight %}

## Import
Now we can import our turtle file.

In a src/cmd-line/core.clj:

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

{% highlight clojure %}
(defonce my-repo (-> "./tmp/grafter-sesame-store2" ses/native-store ses/repo))

(defn import-life-facilities [quads-seq destination]
  (let [now (java.util.Date.)
        quads (->> quads-seq
                   filter-triples
                   (validate-triples (complement has-blank?)))]
    
    (pr/add (ses/rdf-serializer destination) quads)))
{% endhighlight %}

## Conclusion
Everything should work now! Let's just finish the work by [defining a -main function to use the command line.](951_command_line.html)