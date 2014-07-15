---
layout: page
title: Perform row level transformations
---

## How to perform row level transformations

### derive-column
**Derive-column is the most useful function to perform row level transformations** and should be choice #1. This is Grafter philosophy: transform data without modifying source data.
The idea is quite simple: on a row, take one or more cell(s), apply a function, and put the result on a new column.

![Data Screenshot](/assets/210_convert_cell_level_values_0.png)

  derive-column:


![Data Screenshot](/assets/220_row_level_transformations_1.png)

{% highlight clojure %}

user=> (-> '(["Grafter" "rdf" 3])
    (derive-column uriify 2))

(["Grafter" "rdf" 3 "http://www.grafter-is-the-best.com/rdf"])

{% endhighlight %}


![Data Screenshot](/assets/220_row_level_transformations_2.png)


And with a function using more than one argument:

![Data Screenshot](/assets/220_row_level_transformations_3.png)

{% highlight clojure %}

user=> (-> '(["Grafter" "rdf" 3 "foo"])
    (derive-column slug-combine 0 1 2))

(["Grafter" "rdf" 3 "foo" "Grafter/rdf/3"])

{% endhighlight %}

![Data Screenshot](/assets/220_row_level_transformations_4.png)

### swap
Swap is used to


### fuse
