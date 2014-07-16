---
layout: page
title: Command Line
---

## Leiningen project
As a step 0, let's configure a Leiningen project:

{% highlight shell %}
$ lein new cmd-line
$ cd cmd-line
{% endhighlight %}

TOFIX: and in project.clj:

{% highlight clojure %}
(defproject cmd-line "0.1.0-SNAPSHOT"
  :description "FIXME: write description"
  :url "http://example.com/FIXME"
  :license {:name "Eclipse Public License"
            :url "http://www.eclipse.org/legal/epl-v10.html"}
  :dependencies [[org.clojure/clojure "1.6.0"]
                 [grafter "0.1.0"]]
  :main cmd-line.core
  :plugins [[s3-wagon-private "1.1.2"]])
{% endhighlight %}

## Dependencies Prefixers
In a src/cmd-line/prefixers.clj file.

We are going to need every ontologies defined in Grafter, some functions from grafter.parse grafter.protocols and grafter.js. We also need the [Clojure algo.monads API](http://clojure.github.io/algo.monads/) and the clojure string API:

{% highlight clojure %}
(ns cmd-line.prefixers
  (:require [grafter.csv :refer [fuse derive-column parse-csv mapc swap drop-rows _]]
           [grafter.rdf.protocols :as pr]
           [clojure.string :as st]
           [grafter.rdf :refer [prefixer s]]
           [grafter.rdf.ontologies.rdf :refer :all]
           [grafter.rdf.ontologies.void :refer :all]
           [grafter.rdf.ontologies.dcterms :refer :all]
           [grafter.rdf.ontologies.vcard :refer :all]
           [grafter.rdf.ontologies.pmd :refer :all]
           [grafter.rdf.ontologies.qb :refer :all]
           [grafter.rdf.ontologies.os :refer :all]
           [grafter.rdf.ontologies.sdmx-measure :refer :all]
           [grafter.parse :refer [lift-1 blank-m replacer mapper parse-int date-time]]
           [grafter.js :refer [js-fn]]
           [clojure.algo.monads :refer [m-chain m-bind m-result with-monad identity-m]]))
{% endhighlight %}

## Dependencies Pipeline
The whole code will be in a src/cmd-line/pipeline.clj file.

The pipeline function is going to require some Grafter functions:

- parse-csv: simply uses the [Clojure.java.io API reader function](http://clojure.github.io/clojure/clojure.java.io-api.html) to parse our csv file
- drop-rows: drops the first n rows from the CSV
- swap: swaps two columns
- derive-column: adds a new column to the end of the row which is derived from already existing columns
- mapc: takes an array of functions and maps each to the equivalent column position for every row
- fuse: merges columns
- _: identity
- date-time: uses the [clj-time](https://github.com/clj-time/clj-time)'s date-time

And also every prefixies we have defined [at the last step](911_prefixies.html)


{% highlight clojure %}
(ns cmd-line.pipeline
  (:require [grafter.csv :refer [fuse derive-column parse-csv mapc swap drop-rows _]]
            [grafter.parse :refer [date-time]]
            [cmd-line.prefixers :refer :all]))
{% endhighlight %}


# Command Line

## -main

Remember in our project.clj there were this line:

{% highlight clojure %}
:main cmd-line.core
{% endhighlight %}

We have to define a -main function, this function will take input and output as argument:

{% highlight clojure %}
(defn -main [my-csv output]
  (println "About to graft " my-csv)

  (import-life-facilities (make-life-facilities my-csv) output)

  (println my-csv "has been grafted: " output))
{% endhighlight %}

## Test

{% highlight shell %}

$ lein run ./test-data/glasgow-life-facilities.csv glasgow-life-facilities.ttl

About to graft  ./test-data/glasgow-life-facilities.csv
./test-data/glasgow-life-facilities.csv has been grafted:  glasgow-life-facilities.ttl

{% endhighlight %}

![Result!](/assets/951_command_line_1.png)

## Conclusion

What's important in this example is the global process and the global philosophy, but must of all, [the pipeline function](921_pipeline.html) is the key!
