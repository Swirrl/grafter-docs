---
layout: page
title: First steps
---
# First steps

Now that you have Leiningen and a nice environment, you can create a new Leiningen project:

{% highlight shell %}
$ lein new test-graft
$ cd test-graft
{% endhighlight %}

And set the dependencies to use Grafter: in <code>project.clj</code>:

{% highlight clojure %}
(defproject test-graft "0.1.0-SNAPSHOT"
  :description "FIXME: write description"
  :url "http://example.com/FIXME"
  :license {:name "Eclipse Public License"
            :url "http://www.eclipse.org/legal/epl-v10.html"}
  :dependencies [[org.clojure/clojure "1.6.0"]
                 [grafter "0.2.0-SNAPSHOT"]])

{% endhighlight %}

Then you can start developing and working with the Clojure REPL:
{% highlight shell %}
$ lein repl
{% endhighlight %}

Now, you can either check our [HowTo](/howto/index.html) section if you already have something in mind, or go directly to the [Tutorials](/tutorials/index.html) section!
