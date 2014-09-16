---
layout: page
title: Normalise datasets
---

## How to normalise datasets

A fundamental principle of Grafter is that it works with row data - 
Grafter parses rows, transforms and modifies data at a row level and creates graph fragments from rows. All the useful data has to be at the row level!

### Normalization / Denormalization

As an example of this row-oriented process, imagine that a part of your data is in the header (here, the year for each observation):

![normalization](/assets/902_philo_1.png)

Then you probably want to "normalise" the table, which means "put the data into rows".

![normalization](/assets/902_philo_2.png)

Grafter doesn’t natively support normalisation, or reshaping, but because grafter is built on incanter you can do it like this:


{% highlight clojure %}

user=> (require ‘(incanter.core :refer [melt]))

user=> (-> dataset
          (melt column-to-normalise-key)

{% endhighlight %}
