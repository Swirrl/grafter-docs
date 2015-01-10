---
layout: page
title: 3. Understanding Pipelines
---

# Understanding Pipelines

*This is the third part of the Grafter Getting Started Guide.*

Lets take a look at the main file where both of the transformation
pipelines are defined, both the pipe and the graft will be located in
the `pipeline.clj` file in the `./src/<project-name>` directory.

{% highlight clojure %}
(defpipe convert-persons-data
  "Pipeline to convert tabular persons data into a different tabular format."
  [data-file]
  (-> (read-dataset data-file)
      (drop-rows 1)
      (make-dataset [:name :sex :age])
      (derive-column :person-uri [:name] base-id)
      (mapc {:age ->integer
             :sex {"f" (s "female")
                   "m" (s "male")}})))
{% endhighlight %}

## Running Transformations at the Clojure REPL
