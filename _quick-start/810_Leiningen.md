---
layout: page
title: Leiningen
---

# Leiningen

Grafter uses [Leiningen](http://leiningen.org) for project automation and declarative configuration.

> **[grafter "0.1.0"]**


## Download

If you already have Leiningen

You can [download Leiningen](http://leiningen.org) from their website:

Or you can use [HomeBrew](http://brew.sh):

{% highlight shell %}
$ brew install leiningen
{% endhighlight %}

## project

Once you have Leiningen, let's create a new project

{% highlight shell %}
$ lein new test-graft
$ cd test-graft
{% endhighlight %}


Open "project.clj" TOFIX

{% highlight clojure %}
(defproject cmd-line "0.1.0-SNAPSHOT"
  :description "FIXME: write description"
  :url "http://example.com/FIXME"
  :license {:name "Eclipse Public License"
            :url "http://www.eclipse.org/legal/epl-v10.html"}
  :dependencies [[org.clojure/clojure "1.6.0"]
                 [grafter "0.1.0"]]
  :main cmd-line.core
  :plugins [[s3-wagon-private "1.1.2"]])
{% endhighlight %}

{% highlight shell %}
$ lein deps
$ lein repl
{% endhighlight %}
