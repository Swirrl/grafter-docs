---
layout: page
title: The Pipeline function
---

# Pipeline

## Principle
As its name indicates, the pipeline function, aims to link the data we are working on and the graph we want. Basically, pipeline modifies, for each row of the CSV file we are working on, the columns, so we can access or add the exact data we need. 

Behind this there is the Grafter philosophy of transformations and preservation of source data.(FIXME: link_to Transformations and the art of preserving source data)

### Clojure 

#### -> macro

#### let

### Data
For each row the process will be the same so I will just show the two first lines:

![Data Screenshot](/assets/921_pipeline_1.png)

### Dependencies

The pipeline function is going to require some Grafter functions:

- parse-csv: simply uses the [Clojure.java.io API reader function](http://clojure.github.io/clojure/clojure.java.io-api.html) to parse our csv file
- drop-rows: drops the first n rows from the CSV
- swap: swaps two columns
- derive-column: adds a new column to the end of the row which is derived from
already existing columns
- mapc: takes an array of functions and maps each to the equivalent column
position for every row
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

## Step by step process

### Parse CSV and header

First step is to parse the csv and drop the header:

{% highlight clojure %}

(defn pipeline [path-csv] 
 (-> (parse-csv path-csv)
     (drop-rows 1)))
     
{% endhighlight %}

![Drop header Screenshot](/assets/921_pipeline_2.png)


### Date

Then we want to prepare the date, thus we would like to swap the month and the year:

{% highlight clojure %}

(defn pipeline [path-csv] 
 (-> (parse-csv path-csv)
     (drop-rows 1)
     (swap {3 4})))
     
{% endhighlight %}

![Swap Screenshot](/assets/921_pipeline_3.png)

and get

![Swap Screenshot](/assets/921_pipeline_4.png)

###

## Conclusion 