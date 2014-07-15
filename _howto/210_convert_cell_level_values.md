---
layout: page
title: Convert cell level values
---

## How to convert cell level values

### mapc

Mapc function allows you to transform and modify each cell differently.

Let's take a basic CSV:

{% highlight CSV %}

" Grafter ", "csv", 4

{% endhighlight %}

We would like to remove blanks on the first cell (trim function), uriify the second one and let the third one identical.

![Data Screenshot](/assets/210_convert_cell_level_values_1.png)


{% highlight clojure %}
user=> (def uriify (prefixer "http://www.grafter-is-the-best.com/"))

user=> (-> '([" Grafter " "rdf" 3])
    (mapc [trim uriify _]))

(["Grafter" "http://www.grafter-is-the-best.com/rdf" 3])

{% endhighlight %}

![Data Screenshot](/assets/210_convert_cell_level_values_2.png)
