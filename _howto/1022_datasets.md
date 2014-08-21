---
layout: page
title: Handle datasets
---

## How to handle datasets

### open-all-datasets

[open-all-datasets](http://api.grafter.org/0.2/grafter.tabular.html#var-open-all-datasets){:target="_blank"} takes a file path and recursively searches beneath it for dataset-like things, returning a sequence of datasets. In this context a single file can depending on its type be treated as multiple datasets, e.g. an Excel file containing multiple worksheets.

{% highlight clojure %}

user=> (open-all-datasets "./test/grafter/")

{% endhighlight %}


### make-dataset

[make-dataset](http://api.grafter.org/0.2/grafter.tabular.html#var-make-dataset){:target="_blank"} lets you create datasets out of arbitrary data. Consequently it’s mostly a function for developers aiming to target grafter with new file formats.

{% highlight clojure %}

user=> (make-dataset [[1 2 3] ["foo" "bar" "baz"]])

{% endhighlight %}

![ds](/assets/1022_ds_1.png)

By default column names are "a", "b", "c"... But you can define yours:

{% highlight clojure %}

user=> (make-dataset [[1 2 3] ["foo" "bar" "baz"]] ["Grafter" "0.2" "rocks"])

{% endhighlight %}

![ds](/assets/1022_ds_2.png)

You can also call make-dataset on itself, for example to modify the column names:

{% highlight clojure %}

user=> (make-dataset (make-dataset [[1 2 3] ["foo" "bar" "baz"]] ["Grafter" "0.2" "rocks"]) ["PMD" "rocks" "too"])

{% endhighlight %}

![ds](/assets/1022_ds_3.png)

### all-columns

[all-columns](http://api.grafter.org/0.2/grafter.tabular.html#var-all-columns){:target="_blank"} returns a dataset containing just the columns given as arguments (given as name or position):

{% highlight clojure %}

user=> (let [ds (make-dataset [[1 2 3] ["foo" "bar" "baz"]] ["Grafter" "0.2" "rocks"])]
          (all-columns ds ["Grafter" "rocks"]))

{% endhighlight %}

![ds](/assets/1022_ds_4.png)
