---
layout: page
title: Filter rows
---

## How to filter rows

### drop-rows

Drop rows can be useful to remove a header for example. Simply give the number of rows you want to drop as argument.

![Data Screenshot](/assets/230_filter_raws_1.png)

![Data Screenshot](/assets/230_filter_raws_2.png)

Result:

![Data Screenshot](/assets/230_filter_raws_3.png)

{% highlight clojure %}

user=> (-> '( ["Grafter" "rdf" 3]
              ["PMD" "opendata" 2]
              ["Drafter" "data" 1])
            (drop-rows 2))

(["Drafter" "data" 1])


{% endhighlight %}
