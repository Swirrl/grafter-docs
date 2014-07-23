---
layout: page
title: Lookup tables
---

# Miscelaneous Tips & Tricks

## The art of lookup tables

We can imagine different ways to do this - one would be to use SPARQL. Another one is **to build an Hashmap from a table** to be able to pass it a value to lookup something else.

## Grafter's way

{% highlight clojure %}

(coming soon)

{% endhighlight %}

## By hand way

### parse-csv
parse-csv is usually the first step:

{% highlight clojure %}

(:require [grafter.csv :refer [parse-csv]])

{% endhighlight %}


{% highlight clojure %}

user=> (parse-csv csv)
(["Foo" "Bar" "Baz"] ["1" "a" "2014"] ["2" "b" "2014"] ["3" "c" "2014"] ["4" "d" "2014"])

{% endhighlight %}

### hash-map

Build an Hashmap is quite easy:

{% highlight clojure %}

user=>  (def test-lookup
           (->> [["foo" "bar"] ["1" "a"] ["2" "b"]]  ; each vector should have pair number of values
           (into {})))

user=> test-lookup
{"foo" "bar", "1" "a", "2" "b"}

{% endhighlight %}

### map
The map function can be more useful:

{% highlight clojure %}

user=>  (def test-lookup
          (->> (parse-csv csv)
          rest
          (map last)
          set))

user=> test-lookup
#{"2014"}

{% endhighlight %}
