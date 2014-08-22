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

{% highlight clojure %}

(defn pipeline [dataset]
  (-> dataset))

{% endhighlight %}

![Data Screenshot](/assets/921_pipeline_1.png)


### Header
First step is to modify the column names so we can easily work with them and then drop the header.

{% highlight clojure %}

(defn pipeline [dataset]
  (-> dataset
      (make-dataset ["facility-description" "facility-name"
                      "monthly-attendance" "month" "year"
                      "address" "town" "postcode" "website"])
      (drop-rows 1)))

{% endhighlight %}

![Drop header Screenshot](/assets/921_pipeline_2.png)

### Type

Next we would like to get the type of the facility: in this case we want to have: "museums". Instead of modifying source data we will just add a new column with the modified data. Basically we take the column "facility-description" data, apply the function uriify-type, and add the result to a new column called "facility-type":

{% highlight clojure %}

(defn pipeline [dataset]
  (-> dataset
      (make-dataset ["facility-description" "facility-name"
                      "monthly-attendance" "month" "year"
                      "address" "town" "postcode" "website"])
      (drop-rows 1)
      (derive-column "facility-type" ["facility-description"] uriify-type)))


{% endhighlight %}

![Swap Screenshot](/assets/921_pipeline_5.png)

We will define uriify-type function later, here is what it does:

{% highlight clojure %}
user=> (uriify-type "Museums")
"museums"
{% endhighlight %}

![Swap Screenshot](/assets/921_pipeline_6.png)

### Name

We are going to need to have the name in an nice format to make URI. Same way to do:

{% highlight clojure %}

(defn pipeline [dataset]
  (-> dataset
      (make-dataset ["facility-description" "facility-name"
                      "monthly-attendance" "month" "year"
                      "address" "town" "postcode" "website"])
      (drop-rows 1)
      (derive-column "facility-type" ["facility-description"] uriify-type)
      (derive-column "name-slug" ["facility-name"] slugify)))

{% endhighlight %}

![Swap Screenshot](/assets/921_pipeline_7.png)

{% highlight clojure %}

user=> (slugify "Riverside Museum")
"riverside-museum"

{% endhighlight %}

![Swap Screenshot](/assets/921_pipeline_8.png)

### Mapc

This one is a bit different: we will use mapc to apply a different function to each column:

{% highlight clojure %}

(defn pipeline [dataset]
  (-> dataset
      (make-dataset ["facility-description" "facility-name"
                      "monthly-attendance" "month" "year"
                      "address" "town" "postcode" "website"])
      (drop-rows 1)
      (derive-column "facility-type" ["facility-description"] uriify-type)
      (derive-column "name-slug" ["facility-name"] slugify)
      (mapc {"facility-description" uriify-facility
             "monthly-attendance" parse-attendance
             "month" convert-month
             "year" parse-year
             "address" address-line
             "town" city
             "postcode" post-code
             "website" url})))

{% endhighlight %}

![Swap Screenshot](/assets/921_pipeline_9.png)

{% highlight clojure %}

user=> (uriify-facility "Museums")
"http://linked.glasgow.gov.uk/def/urban-assets/Museum"

user=> (parse-attendance 48521)
48521

user=> (parse-year 2013)
2013

user=> (convert-month "September")
9

user=> (address-line "100 Pointhouse Place")
#<sesame$s$reify__448 100 Pointhouse Place> ;; RDF string literals

user=> (city "Glasgow")
#<sesame$s$reify__448 Glasgow> ;; RDF string literals

user=> (post-code "G3 8RS")
#<sesame$s$reify__448 G3 8RS> ;; RDF string literals

user=> (url "http://www.glasgowlife.org.uk/museums/riverside/Pages/default.aspx")
#<URL http://www.glasgowlife.org.uk/museums/riverside/Pages/default.aspx>

{% endhighlight %}

![Swap Screenshot](/assets/921_pipeline_10.png)

It's important to remember that finally, **we want an RDF graph so we have to build, here, RDF type!**

We have changed a lot of things, but each transformation is easy to understand and what is important is that you can apply easily different transofrmations to each column.

### Facilities uri

{% highlight clojure %}

(defn pipeline [dataset]
  (-> dataset
      (make-dataset ["facility-description" "facility-name"
                      "monthly-attendance" "month" "year"
                      "address" "town" "postcode" "website"])
      (drop-rows 1)
      (derive-column "facility-type" ["facility-description"] uriify-type)
      (derive-column "name-slug" ["facility-name"] slugify)
      (mapc {"facility-description" uriify-facility
             "monthly-attendance" parse-attendance
             "month" convert-month
             "year" parse-year
             "address" address-line
             "town" city
             "postcode" post-code
             "website" url})
       (derive-column "ref-facility-uri" ["facility-type" "name-slug"] uriify-refFacility)))

{% endhighlight %}

![Swap Screenshot](/assets/921_pipeline_11.png)

{% highlight clojure %}

user=> (uriify-refFacility "museums" "riverside-museum")
"http://linked.glasgow.gov.uk/id/urban-assets/museums/riverside-museum"

{% endhighlight %}

![Swap Screenshot](/assets/921_pipeline_12.png)

### Postcode

{% highlight clojure %}

(defn pipeline [dataset]
  (-> dataset
      (make-dataset ["facility-description" "facility-name"
                      "monthly-attendance" "month" "year"
                      "address" "town" "postcode" "website"])
      (drop-rows 1)
      (derive-column "facility-type" ["facility-description"] uriify-type)
      (derive-column "name-slug" ["facility-name"] slugify)
      (mapc {"facility-description" uriify-facility
             "monthly-attendance" parse-attendance
             "month" convert-month
             "year" parse-year
             "address" address-line
             "town" city
             "postcode" post-code
             "website" url})
       (derive-column "ref-facility-uri" ["facility-type" "name-slug"] uriify-refFacility)
       (derive-column "postcode-uri" ["postcode"] uriify-pcode)))

{% endhighlight %}

![Swap Screenshot](/assets/921_pipeline_13.png)


![Swap Screenshot](/assets/921_pipeline_14.png)

### Date time

We are now going to create a nice date-time.
First the idea is to swap column "month" and column "year": by doing this we will have the year before the month. This is functionnal programming: order counts!

{% highlight clojure %}

(defn pipeline [dataset]
  (-> dataset
      (make-dataset ["facility-description" "facility-name"
                      "monthly-attendance" "month" "year"
                      "address" "town" "postcode" "website"])
      (drop-rows 1)
      (derive-column "facility-type" ["facility-description"] uriify-type)
      (derive-column "name-slug" ["facility-name"] slugify)
      (mapc {"facility-description" uriify-facility
             "monthly-attendance" parse-attendance
             "month" convert-month
             "year" parse-year
             "address" address-line
             "town" city
             "postcode" post-code
             "website" url})
       (derive-column "ref-facility-uri" ["facility-type" "name-slug"] uriify-refFacility)
       (derive-column "postcode-uri" ["postcode"] uriify-pcode)
       (swap "month" "year")))

{% endhighlight %}

![Swap Screenshot](/assets/921_pipeline_141.png)

![Swap Screenshot](/assets/921_pipeline_142.png)

Then we can derive-column applying date-time:

{% highlight clojure %}

(defn pipeline [dataset]
  (-> dataset
      (make-dataset ["facility-description" "facility-name"
                      "monthly-attendance" "month" "year"
                      "address" "town" "postcode" "website"])
      (drop-rows 1)
      (derive-column "facility-type" ["facility-description"] uriify-type)
      (derive-column "name-slug" ["facility-name"] slugify)
      (mapc {"facility-description" uriify-facility
             "monthly-attendance" parse-attendance
             "month" convert-month
             "year" parse-year
             "address" address-line
             "town" city
             "postcode" post-code
             "website" url})
       (derive-column "ref-facility-uri" ["facility-type" "name-slug"] uriify-refFacility)
       (derive-column "postcode-uri" ["postcode"] uriify-pcode)
       (swap "month" "year")
       (derive-column "date" ["year" "month"] date-time)))

{% endhighlight %}

![Swap Screenshot](/assets/921_pipeline_15.png)

{% highlight clojure %}
user=> (date-time 2013 9)
#<DateTime 2013-09-01T00:00:00.000Z>
{% endhighlight %}

![Swap Screenshot](/assets/921_pipeline_16.png)

### Monthly attendance

Nothing really new from now, we'll keep using derive-column. Here are the two next steps:

{% highlight clojure %}

(defn pipeline [dataset]
  (-> dataset
      (make-dataset ["facility-description" "facility-name"
                      "monthly-attendance" "month" "year"
                      "address" "town" "postcode" "website"])
      (drop-rows 1)
      (derive-column "facility-type" ["facility-description"] uriify-type)
      (derive-column "name-slug" ["facility-name"] slugify)
      (mapc {"facility-description" uriify-facility
             "monthly-attendance" parse-attendance
             "month" convert-month
             "year" parse-year
             "address" address-line
             "town" city
             "postcode" post-code
             "website" url})
       (derive-column "ref-facility-uri" ["facility-type" "name-slug"] uriify-refFacility)
       (derive-column "postcode-uri" ["postcode"] uriify-pcode)
       (swap "month" "year")
       (derive-column "date" ["year" "month"] date-time)
       (derive-column "prefix-date" ["date"] prefix-monthly-attendance)
       (derive-column "type-name" ["facility-type" "name-slug"] slug-combine)))

{% endhighlight %}

![Swap Screenshot](/assets/921_pipeline_17.png)

![Swap Screenshot](/assets/921_pipeline_18.png)

### Last step!

{% highlight clojure %}

(defn pipeline [dataset]
  (-> dataset
      (make-dataset ["facility-description" "facility-name"
                      "monthly-attendance" "month" "year"
                      "address" "town" "postcode" "website"])
      (drop-rows 1)
      (derive-column "facility-type" ["facility-description"] uriify-type)
      (derive-column "name-slug" ["facility-name"] slugify)
      (mapc {"facility-description" uriify-facility
             "monthly-attendance" parse-attendance
             "month" convert-month
             "year" parse-year
             "address" address-line
             "town" city
             "postcode" post-code
             "website" url})
       (derive-column "ref-facility-uri" ["facility-type" "name-slug"] uriify-refFacility)
       (derive-column "postcode-uri" ["postcode"] uriify-pcode)
       (swap "month" "year")
       (derive-column "date" ["year" "month"] date-time)
       (derive-column "prefix-date" ["date"] prefix-monthly-attendance)
       (derive-column "type-name" ["facility-type" "name-slug"] slug-combine)
       (derive-column "observation-uri" ["prefix-date" "type-name"] str)))

{% endhighlight %}

![Swap Screenshot](/assets/921_pipeline_19.png)


![Swap Screenshot](/assets/921_pipeline_20.png)

## Conclusion

We now have a much more useful set of data and we are going to be able to use it directly in the next part of this overview with [the creation of the graph fragments.](907_graph.html)
