---
layout: page
title: URI patterns
---

# Data modelling

## URI patterns and the art of the URI

Good URIs are the condition sine qua none of a useful and clear RDFization.

### patterns

Our recommendation would be use, when possible, the more classic URI pattern :

![uri pattern](/assets/410_uri_patterns_1.png)

### prefixer

The prefixer function is really useful - and really simple to understand / use!

{% highlight clojure %}

grafter.rdf=> (def my-app-uri
                (prefixer "http://www.swirrl.com/"))


grafter.rdf=> (my-app-uri "grafter")

"http://www.swirrl.com/grafter"

{% endhighlight %}

### uriifying

There are more powerful ways to create URIs:

{% highlight clojure %}

grafter.rdf=> (defn uriify-my-app [name version]
                (str (my-app-uri name) "/" version))

grafter.rdf=> (uriify-my-app "grafter" "1")

"http://www.swirrl.com/grafter/1"

{% endhighlight %}

### slugify

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
