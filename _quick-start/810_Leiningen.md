---
layout: page
title: Leiningen
---

# Leiningen

## Download
You can [download Leiningen](http://leiningen.org) from their website, or use [HomeBrew](http://brew.sh):
{% highlight shell %}
$ brew install leiningen
{% endhighlight %}


## Grafter
You can simply use Grafter in a Leiningen project by adding it to your <code>:dependencies</code> in <code>project.clj</code>:

{% highlight clojure %}

:dependencies [grafter "0.2.0-SNAPSHOT"]

{% endhighlight %}
