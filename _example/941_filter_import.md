---
layout: page
title: 5. Filter and import
---
# Filter and import

It's now time to import those quads into a real graph or a Turtle file. However if we try to import the data in its current state it will fail due to some missing URIs. Grafter validates the triples before they are written, and it does not accept missing data. Therefore we have to filter out the triples with bad data so we do not attempt to write them.

## Filter

### filter-triples
When checking our dataset we can see that there is some missing data in the columns

- Postcode
- Website

We want to remove the triples with bad data in either of these columns.

{% highlight clojure %}

(defn filter-triples [triples]
  (filter #(not (and (#{vcard:postal-code os:postcode vcard:hasUrl} (pr/predicate %1))
                     (blank? (pr/object %1)))) triples))

{% endhighlight %}

## Import
Now we can import our turtle file.

{% highlight clojure %}

(defn import-life-facilities
  [quads-seq destination]
  (let [now (java.util.Date.)
        quads (->> quads-seq
                   filter-triples
                   (validate-triples (complement has-blank?)))]

    (pr/add (ses/rdf-serializer destination) quads)))

(defn -main [path output]
  (-> (open-all-datasets path)
      first
      pipeline
      glasgow-life-facilities-template
      (import-life-facilities output)))

{% endhighlight %}

## Conclusion
The import should now complete successfully. Finally, we'll [define a Leiningen project](951_command_line.html) so we can graft from the command line.
