---
layout: page
title: URI patterns
---

# Data modelling

## URI patterns and the art of the URI

Good URIs are the a fundamental requirement for an RDF dataset.

### Patterns

Our recommendation would be use, when possible, the more classic URI pattern :

![uri pattern](/assets/410_uri_patterns_1.png)

### prefixer

[prefixer](http://api.grafter.org/0.2/grafter.rdf.html#var-prefixer){:target="_blank"} is a simple way of constructing a URI from a prefix.

{% highlight clojure %}

user=> (def my-app-uri
         (prefixer "http://www.swirrl.com/"))

user=> (my-app-uri "grafter")
"http://www.swirrl.com/grafter"

{% endhighlight %}

### Uriifying

There are some powerful ways to create URIs:

{% highlight clojure %}

user=> (defn uriify-my-app
         [name version]
         (str (my-app-uri name) "/" version))

user=> (uriify-my-app "grafter" "1")
"http://www.swirrl.com/grafter/1"

{% endhighlight %}

### Slugifying

{% highlight clojure %}

user=> (require '[clojure.string :as st])

user=> (defn slugify [string]
         (-> string
             st/trim
             (st/lower-case)
             (st/replace " " "-")))

user=> (slugify " Foo bAr")
"foo-bar"

{% endhighlight %}
