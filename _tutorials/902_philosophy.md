---
layout: page
title: Grafter's data philosophy
---

# Grafter's data philosophy

## Rows, rows, rows

If there were one thing to understand from the beginning that would be **Grafter works with rows!**
Indeed, Grafter parses rows, transform and modify data at a row level and create graph fragments from rows. All the useful data have to be at the row level!

## Normalization / Denormalization

As an example of this rows oriented process, imagine that a part of your data are in the header (here, the year for each observation):

![normalization](/assets/902_philo_1.png)

Then you probably want to "normalize" the table, which means "put the data into rows".

![normalization](/assets/902_philo_2.png)

One way to do this is using the Grafter's "normalize" function:

{% highlight clojure %}

grafter.csv=> (def csv
                 [["grafter version" "release" "grafted csv 2014" "grafted csv 2015" "grafted csv 2016"]
                  [1 2014 50000 200000 25000]
                  [2 2015 300000 700000 1000000]])

grafter.csv=> (normalise csv [2 3 4])

( [1 2014 "grafted csv 2014" 50000]
  [1 2014 "grafted csv 2015" 200000]
  [1 2014 "grafted csv 2016" 25000]
  [2 2015 "grafted csv 2014" 300000]
  [2 2015 "grafted csv 2015" 700000]
  [2 2015 "grafted csv 2016" 1000000])

{% endhighlight %}
