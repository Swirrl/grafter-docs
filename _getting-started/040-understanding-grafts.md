---
layout: page
title: 4. Understanding Grafts
---

# Understanding Grafts

*This is the fourth part of the Grafter Getting Started Guide*

Lets continue with the main file where both the `pipe`and `graft` transformation pipelines are defined.

Lets focus on the `defgraft` definition:

{% highlight clojure %}

(defgraft convert-persons-data-to-graph
  "Pipeline to convert the tabular persons data sheet into graph data."
  convert-persons-data make-graph)

{% endhighlight %}

While `defpipe` forms are syntactically identical to Clojure's `defn` form, `defgraft` forms are syntactically identical to Clojure's `def` form. Indeed, `defgraft` creates a var with the composition of the functions given as arguments and advertises that var to the plugin and other Grafter services.

For example if you run the command `lein grafter list grafts` the plugin
will search the projects classpath for any clj files with valid
defgraft definitions, and list them.

`defgraft` is necessary as it allows Grafter to distinguish `grafts` from ordinary functions.

## What is a graft?

A graft is any function defined with Grafters `defgraft`, that can convert 0 or more `datasettable` arguments into an `RDF` graph output.



## Running Transformations at the Clojure REPL
