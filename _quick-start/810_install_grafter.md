---
layout: page
title: Install
---

# Install Grafter

## Download Leiningen
Leiningen is the most well supported build tool for Clojure development and it's what we use for Grafter and our examples.

You will need to download and [install Leiningen](http://leiningen.org) from their website to follow this quick start guide.

## Add Grafter to dependencies
You can simply use Grafter in a Leiningen project by adding it to your <code>:dependencies</code> in <code>project.clj</code>:

{% highlight clojure %}
  :dependencies [grafter "0.2.0-SNAPSHOT"]
{% endhighlight %}
