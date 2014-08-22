---
layout: page
title: Filter and import
---
# Filter and import

It's now time to import those quads into a real graph on a Turtle file. BUT if we try now it won't work because of some blanks in our dataset (some missing url basically). When Grafter writes the triples, it validates them and Grafter does not accept missing data. Thus we have to filter those triples so that, if there is a missing data, the whole triple is not written.

## Filter

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
Everything should work now! Let's just finish the work by [defining a Leiningen project to wrap all those code bits and to graft from the command line.](951_command_line.html)
