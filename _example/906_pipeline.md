---
layout: page
title: 2. Define a Pipeline
---

# Pipeline

The pipeline function links the data we are working on with the graph we want. It transforms the set of columns in the source table, so we can access or add the exact data we need.

This adheres to the Grafter philosophy of [transformations and preservation of source data.](/howto/101_transformations.html)

Early in this tutorial, some functions are declared without their definition being given. Example usages of these functions are given to illustrate
their meaning, however they only appear as part of a larger example so a full understanding is not required.

### Data
For each row the process will be the same so I will just show the two first lines:

{% highlight clojure %}

(defn pipeline [dataset]
  (-> dataset))

{% endhighlight %}

![Data Screenshot](/assets/921_pipeline_1.png)


### Header
First step is to specify the column names so we can easily work with them and then drop the header.

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

Next we would like to get the type of the facility: in this case we want to have: "museums". Instead of modifying source data we will just add a new column with the modified data. Basically we take the column "facility-description" data, apply the function clean-type, and add the result to a new column called "facility-type":

{% highlight clojure %}

(defn pipeline [dataset]
  (-> dataset
      (make-dataset ["facility-description" "facility-name"
                      "monthly-attendance" "month" "year"
                      "address" "town" "postcode" "website"])
      (drop-rows 1)
      (derive-column "facility-type" ["facility-description"] clean-type)))


{% endhighlight %}

![Swap Screenshot](/assets/921_pipeline_5.png)

We will define clean-type function later, here is what it does:

{% highlight clojure %}
user=> (clean-type "Museums")
"museums"
{% endhighlight %}

![Swap Screenshot](/assets/921_pipeline_6.png)

### Name

We require the name in a format suitable for inclusion in a URI.

{% highlight clojure %}

(defn pipeline [dataset]
  (-> dataset
      (make-dataset ["facility-description" "facility-name"
                      "monthly-attendance" "month" "year"
                      "address" "town" "postcode" "website"])
      (drop-rows 1)
      (derive-column "facility-type" ["facility-description"] clean-type)
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
      (derive-column "facility-type" ["facility-description"] clean-type)
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

It's important to remember that ultimately we want an RDF graph so we have to build an RDF type.

We have changed a lot of things, but each transformation is easy to understand and you can apply different transformations to each column.

### Facilities uri

{% highlight clojure %}

(defn pipeline [dataset]
  (-> dataset
      (make-dataset ["facility-description" "facility-name"
                      "monthly-attendance" "month" "year"
                      "address" "town" "postcode" "website"])
      (drop-rows 1)
      (derive-column "facility-type" ["facility-description"] clean-type)
      (derive-column "name-slug" ["facility-name"] slugify)
      (mapc {"facility-description" uriify-facility
             "monthly-attendance" parse-attendance
             "month" convert-month
             "year" parse-year
             "address" address-line
             "town" city
             "postcode" post-code
             "website" url})
       (derive-column "ref-facility-uri" ["facility-type" "name-slug"] uriify-ref-facility)))

{% endhighlight %}

![Swap Screenshot](/assets/921_pipeline_11.png)

{% highlight clojure %}

user=> (uriify-ref-facility "museums" "riverside-museum")
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
      (derive-column "facility-type" ["facility-description"] clean-type)
      (derive-column "name-slug" ["facility-name"] slugify)
      (mapc {"facility-description" uriify-facility
             "monthly-attendance" parse-attendance
             "month" convert-month
             "year" parse-year
             "address" address-line
             "town" city
             "postcode" post-code
             "website" url})
       (derive-column "ref-facility-uri" ["facility-type" "name-slug"] uriify-ref-facility)
       (derive-column "postcode-uri" ["postcode"] uriify-pcode)))

{% endhighlight %}

![Swap Screenshot](/assets/921_pipeline_13.png)


![Swap Screenshot](/assets/921_pipeline_14.png)

### Date time

We are now going to create a date column.
First the idea is to swap column "month" and column "year": by doing this we will have the year before the month - remember column order is significant!

{% highlight clojure %}

(defn pipeline [dataset]
  (-> dataset
      (make-dataset ["facility-description" "facility-name"
                      "monthly-attendance" "month" "year"
                      "address" "town" "postcode" "website"])
      (drop-rows 1)
      (derive-column "facility-type" ["facility-description"] clean-type)
      (derive-column "name-slug" ["facility-name"] slugify)
      (mapc {"facility-description" uriify-facility
             "monthly-attendance" parse-attendance
             "month" convert-month
             "year" parse-year
             "address" address-line
             "town" city
             "postcode" post-code
             "website" url})
       (derive-column "ref-facility-uri" ["facility-type" "name-slug"] uriify-ref-facility)
       (derive-column "postcode-uri" ["postcode"] uriify-pcode)
       (swap "month" "year")))

{% endhighlight %}

![Swap Screenshot](/assets/921_pipeline_141.png)

![Swap Screenshot](/assets/921_pipeline_142.png)

Then we can create a new column applying the date-time function.

{% highlight clojure %}

(defn pipeline [dataset]
  (-> dataset
      (make-dataset ["facility-description" "facility-name"
                      "monthly-attendance" "month" "year"
                      "address" "town" "postcode" "website"])
      (drop-rows 1)
      (derive-column "facility-type" ["facility-description"] clean-type)
      (derive-column "name-slug" ["facility-name"] slugify)
      (mapc {"facility-description" uriify-facility
             "monthly-attendance" parse-attendance
             "month" convert-month
             "year" parse-year
             "address" address-line
             "town" city
             "postcode" post-code
             "website" url})
       (derive-column "ref-facility-uri" ["facility-type" "name-slug"] uriify-ref-facility)
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

Nothing new here - we'll keep using derive-column. Here are the two next steps:

{% highlight clojure %}

(defn pipeline [dataset]
  (-> dataset
      (make-dataset ["facility-description" "facility-name"
                      "monthly-attendance" "month" "year"
                      "address" "town" "postcode" "website"])
      (drop-rows 1)
      (derive-column "facility-type" ["facility-description"] clean-type)
      (derive-column "name-slug" ["facility-name"] slugify)
      (mapc {"facility-description" uriify-facility
             "monthly-attendance" parse-attendance
             "month" convert-month
             "year" parse-year
             "address" address-line
             "town" city
             "postcode" post-code
             "website" url})
       (derive-column "ref-facility-uri" ["facility-type" "name-slug"] uriify-ref-facility)
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
      (derive-column "facility-type" ["facility-description"] clean-type)
      (derive-column "name-slug" ["facility-name"] slugify)
      (mapc {"facility-description" uriify-facility
             "monthly-attendance" parse-attendance
             "month" convert-month
             "year" parse-year
             "address" address-line
             "town" city
             "postcode" post-code
             "website" url})
       (derive-column "ref-facility-uri" ["facility-type" "name-slug"] uriify-ref-facility)
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

We now have a much more useful set of data and we are going to be able to use it directly in the next part of this tutorial with [the creation of the graph fragments.](907_graph.html)
