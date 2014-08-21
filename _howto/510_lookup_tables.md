---
layout: page
title: Lookup tables
---

# Miscelaneous Tips & Tricks

## The art of lookup tables

Let's say you are working on a dataset and you need some data from another dataset. You need what we call a lookup table. Build-lookup-table returns a function that goes pick the data you want in the other dataset.

### build-lookup-table

[build-lookup-table](http://api.grafter.org/0.2/grafter.tabular.html#var-build-lookup-table){:target="_blank"} takes as argument a dataset, one or several key columns and a value column. It returns a function which takes some key values as argument and returns the corresponding value.

We can then pass to this function some key values and it returns the corresponding data in the value column.

![Data Screenshot](/assets/510_lookup_table_2.png)

### Usage

{% highlight clojure %}

user=> (let [lookup (build-lookup-table dataset2 ["name"] "comment")]
          (-> dataset1
              (derive-column "comment" ["name"] lookup)))

{% endhighlight %}

![Data Screenshot](/assets/510_lookup_table_3.png)
