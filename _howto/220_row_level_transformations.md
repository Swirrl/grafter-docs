---
layout: page
title: Perform row level transformations
---

## How to perform row level transformations

### derive-column

[derive-column](http://api.grafter.org/0.2/grafter.tabular.html#var-derive-column){:target="_blank"} is the primary function for performing row-level transformations.
The idea is quite simple: on a row, apply a function to one or more cells and put the result in a new column.

![Data Screenshot](/assets/210_convert_cell_level_values_0.png)

{% highlight clojure %}

user=> (derive-column dataset "d" ["b"] uriify)

{% endhighlight %}

![Data Screenshot](/assets/220_row_level_transformations_1.png)


![Data Screenshot](/assets/220_row_level_transformations_2.png)


And with a function using more than one argument:

{% highlight clojure %}

user=> (derive-column dataset "new-column-name" ["a" "b" "c"] slug-combine)

{% endhighlight %}

![Data Screenshot](/assets/220_row_level_transformations_3.png)


![Data Screenshot](/assets/220_row_level_transformations_4.png)

Note that column order is significant:

{% highlight clojure %}

user=> (derive-column dataset "new-column-name" ["b" "c" "a"] slug-combine)

{% endhighlight %}

![Data Screenshot](/assets/220_row_level_transformations_42.png)

Since column order is important, there is a function to change the columns order if needed: swap.

### swap

[swap](http://api.grafter.org/0.2/grafter.tabular.html#var-swap){:target="_blank"} is used to swap columns.

{% highlight clojure %}

user=> (swap dataset "b" "d")

{% endhighlight %}

![Data Screenshot](/assets/220_row_level_transformations_5.png)


![Data Screenshot](/assets/220_row_level_transformations_6.png)

You can apply a chain of swap operations by providing a sequence of swap pairs:

{% highlight clojure %}

user=> (swap dataset "b" "d" "d" "a")

{% endhighlight %}

![Data Screenshot](/assets/220_row_level_transformations_7.png)


![Data Screenshot](/assets/220_row_level_transformations_8.png)


![Data Screenshot](/assets/220_row_level_transformations_9.png)
