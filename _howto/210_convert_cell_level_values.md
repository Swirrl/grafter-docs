---
layout: page
title: Convert cell level values
---
# Row & Cell level

## How to convert cell level values

### mapc

The [mapc](http://api.grafter.org/0.2/grafter.tabular.html#var-mapc){:target="_blank"} function allows you to transform and modify each cell differently.

Let's take a basic dataset:

![Data Screenshot](/assets/210_convert_cell_level_values_0.png)


We would like to remove whitespace around the values in the first column (trim function), uriify the second and leave the third column unmodified.

![Data Screenshot](/assets/210_convert_cell_level_values_1.png)


{% highlight clojure %}

user=> (def uriify (prefixer "http://www.grafter-is-the-best.com/"))

user=> (mapc dataset {"A" trim, "B" uriify, "C" identity})

{% endhighlight %}

![Data Screenshot](/assets/210_convert_cell_level_values_2.png)
