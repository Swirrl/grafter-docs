---
layout: page
title: Normalization / Denormalization
---

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

(coming soon in Grafter)

{% endhighlight %}

Waiting for it to arrive in Grafter, you may want to try [Incanter/melt](http://liebke.github.io/incanter/)
