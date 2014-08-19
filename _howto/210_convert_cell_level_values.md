---
layout: page
title: Convert cell level values
---

## How to convert cell level values

### mapc

Mapc function allows you to transform and modify each cell differently.

Let's take a basic dataset:

![Data Screenshot](/assets/210_convert_cell_level_values_0.png)


We would like to remove blanks on the first cell (trim function), uriify the second one and let the third one identical.

![Data Screenshot](/assets/210_convert_cell_level_values_1.png)


{% highlight clojure %}
user=> (def uriify (prefixer "http://www.grafter-is-the-best.com/"))

user=> (mapc dataset {"A" trim, "B" uriify, "C" identity})

{% endhighlight %}

![Data Screenshot](/assets/210_convert_cell_level_values_2.png)
