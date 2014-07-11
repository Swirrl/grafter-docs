---
layout: page
title: Command Line
---

# Command Line

## -main

Remember in our project.clj there were this line:

{% highlight clojure %}
:main cmd-line.core
{% endhighlight %}

We have to define a -main function, this function will take input and output as argument:

{% highlight clojure %}
(defn -main [my-csv output]
  (println "About to graft " my-csv)
  
  (import-life-facilities (make-life-facilities my-csv) output)
  
  (println my-csv "has been grafted: " output))
{% endhighlight %}

## Test

{% highlight shell %}

$ lein run ./test-data/glasgow-life-facilities.csv glasgow-life-facilities.ttl

About to graft  ./test-data/glasgow-life-facilities.csv
./test-data/glasgow-life-facilities.csv has been grafted:  glasgow-life-facilities.ttl

{% endhighlight %}

![Result!](/assets/951_command_line_1.png)

## Conclusion

What's important in this example is the global process and the global philosophy, but must of all, [the pipeline function](921_pipeline.html) is the key! 