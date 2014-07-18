---
layout: page
title: Leiningen
---

# Leiningen

You can simply use Grafter in a [Leiningen](http://leiningen.org) project by adding <code> :dependencies <strong> [grafter "0.1.0"] </strong></code> to <code> project.clj </code>

## Download
You can [download Leiningen](http://leiningen.org) from their website:

Or using [HomeBrew](http://brew.sh):
{% highlight shell %}
$ brew install leiningen
{% endhighlight %}

## project

Once you have Leiningen, create a new project:

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
