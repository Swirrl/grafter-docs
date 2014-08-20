---
layout: page
title: Perform row level transformations
---

## How to perform row level transformations

### derive-column
**Derive-column is the most useful function to perform row level transformations** and should be choice #1. This is Grafter philosophy: transform data without modifying source data.
The idea is quite simple: on a row, take one or more cell(s), apply a function, and put the result on a new column.

![Data Screenshot](/assets/210_convert_cell_level_values_0.png)

<code>user=> (<a href="http://api.grafter.org/0.2/grafter.tabular.html#var-derive-column" target="_blank">derive-column</a> dataset "d" ["b"] uriify)</code>

![Data Screenshot](/assets/220_row_level_transformations_1.png)


![Data Screenshot](/assets/220_row_level_transformations_2.png)


And with a function using more than one argument:

{% highlight clojure %}

user=> ( derive-column dataset "new-column-name" ["a" "b" "c"] slug-combine)

{% endhighlight %}

![Data Screenshot](/assets/220_row_level_transformations_3.png)


![Data Screenshot](/assets/220_row_level_transformations_4.png)

Note that **order count:**

{% highlight clojure %}

user=> (derive-column dataset "new-column-name" ["b" "c" "a"] slug-combine)

{% endhighlight %}

![Data Screenshot](/assets/220_row_level_transformations_42.png)

And as order count, there is a function to change the columns order if needed, swap.

### swap
Swap is used to swap columns.

{% highlight clojure %}

user=> (swap dataset "b" "d")

{% endhighlight %}

![Data Screenshot](/assets/220_row_level_transformations_5.png)


![Data Screenshot](/assets/220_row_level_transformations_6.png)

And you can combine swaps adding an even number of columns :

{% highlight clojure %}

user=> (swap dataset "b" "d" "d" "a")

{% endhighlight %}

![Data Screenshot](/assets/220_row_level_transformations_7.png)


![Data Screenshot](/assets/220_row_level_transformations_8.png)


![Data Screenshot](/assets/220_row_level_transformations_9.png)
