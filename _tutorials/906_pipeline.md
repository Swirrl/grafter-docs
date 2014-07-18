---
layout: page
title: The Pipeline function
---

# Pipeline

As its name indicates, the pipeline function, aims to link the data we are working on and the graph we want. Basically, pipeline modifies, for each row of the CSV file we are working on, the columns, so we can access or add the exact data we need.

Behind this there is the Grafter philosophy of [transformations and preservation of source data.](/howto/101_transformations.html)

We are going to use some functions and prefixes that are defined later in this tutorial, but that are "less interesting". When I can a give some examples so you can understand everything and focus on the way Pipeline works. This part and the Create Graph part are the most important, others are just "making things work".

### Data
For each row the process will be the same so I will just show the two first lines:

![Data Screenshot](/assets/921_pipeline_1.png)


### Parse CSV and header
First step is to parse the csv and drop the header:

{% highlight clojure %}
(defn pipeline [path-csv]
 (-> (parse-csv path-csv)
     (drop-rows 1)))
{% endhighlight %}

![Drop header Screenshot](/assets/921_pipeline_2.png)

### Type

Next we would like to get the type: in this case we want to have: "museums". Instead of modifying source data we will just add a new column with the modified data. Basically we take the column 0 data, apply the function uriify-type, and add the result to a new column:

{% highlight clojure %}
(defn pipeline [path-csv]
 (-> (parse-csv path-csv)
     (drop-rows 1)
     (derive-column uriify-type 0)))
{% endhighlight %}

![Swap Screenshot](/assets/921_pipeline_5.png)

We will define uriify-type function later, here is what it does:

{% highlight clojure %}
cmd-line.prefixers=> (uriify-type "Museums")
"museums"
{% endhighlight %}

![Swap Screenshot](/assets/921_pipeline_6.png)

### Name

We are going to need to have the name in an nice format to make URI. Same way to do:

{% highlight clojure %}
(defn pipeline [path-csv]
 (-> (parse-csv path-csv)
     (drop-rows 1)
     (derive-column uriify-type 0)
     (derive-column slugify-facility 1)))
{% endhighlight %}

![Swap Screenshot](/assets/921_pipeline_7.png)

{% highlight clojure %}
cmd-line.prefixers=> (slugify-facility "Riverside Museum")
"riverside-museum"
{% endhighlight %}

![Swap Screenshot](/assets/921_pipeline_8.png)

### Mapc

This one is a bit different: we will use mapc to apply a different function to each column:

{% highlight clojure %}
(defn pipeline [path-csv]
 (-> (parse-csv path-csv)
     (drop-rows 1)
     (derive-column uriify-type 0)
     (derive-column slugify-facility 1)
     (mapc [uriify-facility trim parse-attendance parse-year convert-month address-line city post-code url _ _])))
{% endhighlight %}

![Swap Screenshot](/assets/921_pipeline_9.png)

{% highlight clojure %}
cmd-line.prefixers=> (uriify-facility "Museums")
"http://linked.glasgow.gov.uk/def/urban-assets/Museum"
cmd-line.prefixers=> (trim "Riverside Museum")
"Riverside Museum"
cmd-line.prefixers=> (parse-attendance 48521)
48521
cmd-line.prefixers=> (parse-year 2013)
2013
cmd-line.prefixers=> (convert-month "September")
9
cmd-line.prefixers=> (address-line "100 Pointhouse Place")
#<sesame$s$reify__448 100 Pointhouse Place>
cmd-line.prefixers=> (city "Glasgow")
#<sesame$s$reify__448 Glasgow>
cmd-line.prefixers=> (post-code "G3 8RS")
#<sesame$s$reify__448 G3 8RS>
md-line.prefixers=> (url "http://www.glasgowlife.org.uk/museums/riverside/Pages/default.aspx")
#<URL http://www.glasgowlife.org.uk/museums/riverside/Pages/default.aspx>
cmd-line.prefixers=> (_ "museums")
"museums"
{% endhighlight %}

![Swap Screenshot](/assets/921_pipeline_10.png)

It's important to remember that finally, **we want an RDF graph so we have to build, here, RDF type!**

We have changed a lot of things, but each transformation is easy to understand and what is important is that you can apply easily different transofrmations to each column.

### Facilities uri

{% highlight clojure %}
(defn pipeline [path-csv]
 (-> (parse-csv path-csv)
     (drop-rows 1)
     (derive-column uriify-type 0)
     (derive-column slugify-facility 1)
     (mapc [uriify-facility trim parse-attendance parse-year convert-month address-line city post-code url _ _])
     (derive-column uriify-refFacility 9 10)))
{% endhighlight %}

![Swap Screenshot](/assets/921_pipeline_11.png)

{% highlight clojure %}
cmd-line.prefixers=> (uriify-refFacility "museums" "riverside-museum")
"http://linked.glasgow.gov.uk/id/urban-assets/museums/riverside-museum"
{% endhighlight %}

![Swap Screenshot](/assets/921_pipeline_12.png)

### Postcode

{% highlight clojure %}
(defn pipeline [path-csv]
 (-> (parse-csv path-csv)
     (drop-rows 1)
     (derive-column uriify-type 0)
     (derive-column slugify-facility 1)
     (mapc [uriify-facility trim parse-attendance parse-year convert-month address-line city post-code url _ _])
     (derive-column uriify-refFacility 9 10)
     (derive-column uriify-pcode 7)))
{% endhighlight %}

![Swap Screenshot](/assets/921_pipeline_13.png)


![Swap Screenshot](/assets/921_pipeline_14.png)

### Date time

We are now going to create a nice date-time.
First the idea is to swap column 3 and column 4: by doing this we will have the year before the month. This is functionnal programming: order counts!

{% highlight clojure %}
(defn pipeline [path-csv]
 (-> (parse-csv path-csv)
     (drop-rows 1)
     (derive-column uriify-type 0)
     (derive-column slugify-facility 1)
     (mapc [uriify-facility trim parse-attendance parse-year convert-month address-line city post-code url _ _])
     (derive-column uriify-refFacility 9 10)
     (derive-column uriify-pcode 7)
     (swap {3 4})))
{% endhighlight %}

![Swap Screenshot](/assets/921_pipeline_141.png)

![Swap Screenshot](/assets/921_pipeline_142.png)

Then we can apply date-time, the function takes two arguments, on column 3 and 4, fuse take the result of the function and put it in the first column concerned (here 3) and delete the second one:

{% highlight clojure %}
(defn pipeline [path-csv]
 (-> (parse-csv path-csv)
     (drop-rows 1)
     (derive-column uriify-type 0)
     (derive-column slugify-facility 1)
     (mapc [uriify-facility trim parse-attendance parse-year convert-month address-line city post-code url _ _])
     (derive-column uriify-refFacility 9 10)
     (derive-column uriify-pcode 7)
     (swap {3 4})
     (fuse date-time 3 4)))
{% endhighlight %}

![Swap Screenshot](/assets/921_pipeline_15.png)

{% highlight clojure %}
cmd-line.prefixers=> (date-time 2013 9)
#<DateTime 2013-09-01T00:00:00.000Z>
{% endhighlight %}

![Swap Screenshot](/assets/921_pipeline_16.png)

Nothing really new from now, but still interesting!

### Monthly attendance

{% highlight clojure %}
(defn pipeline [path-csv]
 (-> (parse-csv path-csv)
     (drop-rows 1)
     (derive-column uriify-type 0)
     (derive-column slugify-facility 1)
     (mapc [uriify-facility trim parse-attendance parse-year convert-month address-line city post-code url _ _])
     (derive-column uriify-refFacility 9 10)
     (derive-column uriify-pcode 7)
     (swap {3 4})
     (fuse date-time 3 4)
     (derive-column prefix-monthly-attendance 3)))
{% endhighlight %}

![Swap Screenshot](/assets/921_pipeline_17.png)


![Swap Screenshot](/assets/921_pipeline_18.png)

### Last step!

Actually we combine two steps here but it should be two really easy steps now:

{% highlight clojure %}
(defn pipeline [path-csv]
 (-> (parse-csv path-csv)
     (drop-rows 1)
     (derive-column uriify-type 0)
     (derive-column slugify-facility 1)
     (mapc [uriify-facility trim parse-attendance parse-year convert-month address-line city post-code url _ _])
     (derive-column uriify-refFacility 9 10)
     (derive-column uriify-pcode 7)
     (swap {3 4})
     (fuse date-time 3 4)
     (derive-column prefix-monthly-attendance 3)
     (derive-column slug-combine 8 9)
     (fuse str 12 13)))
{% endhighlight %}

![Swap Screenshot](/assets/921_pipeline_19.png)

{% highlight clojure %}
cmd-line.prefixers=> (slug-combine "museums" "riverside-museum")
"museums/riverside-museum"
cmd-line.prefixers=> (str "http://linked.glasgow.gov.uk/data/glasgow-life-attendances/2013-9/" "museums/riverside-museum")
"http://linked.glasgow.gov.uk/data/glasgow-life-attendances/2013-9/museums/riverside-museum"
{% endhighlight %}

![Swap Screenshot](/assets/921_pipeline_20.png)

## Conclusion

We now have a much more useful set of data and we are going to be able to use it directly in the next part of this overview with [the creation of the graph fragments.](907_graph.html)
