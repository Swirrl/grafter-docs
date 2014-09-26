---
layout: page
title: Filter rows
---

## How to filter rows

### drop-rows

[drop-rows](http://api.grafter.org/0.2/grafter.tabular.html#var-drop-rows){:target="_blank"} can be used to remove some rows. Simply give the number of rows you want to drop as argument.

{% highlight clojure %}

user=> (drop-rows dataset 2)

{% endhighlight %}

![Data Screenshot](/assets/230_filter_raws_1.png)

![Data Screenshot](/assets/230_filter_raws_2.png)

![Data Screenshot](/assets/230_filter_raws_3.png)
