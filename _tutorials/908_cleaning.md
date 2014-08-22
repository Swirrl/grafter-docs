---
layout: page
title: Data cleaning
---
# Data cleaning

This is almost a preparation for data cleaning: we are going to define what will be useful for our next data transformations. We are going to need to remove blanks around a string, to create RDF types, to replace comma or blanks, to parse, to convert and so on...


## Monads

If you don't know about monad yet I recommand [this explanation](http://onclojure.com/2009/03/05/a-monad-tutorial-for-clojure-programmers-part-1/) and/or [the documentation](http://clojure.github.io/algo.monads/)

Here are every definition that we will have to use:

{% highlight clojure %}

(def clean-type {"Museums" "museums"
                  "Arts" "arts-centres"
                  "Community Facility" "community-facilities"
                  "Libraries" "libraries"
                  "Music" "music-venues"
                  "Sport Centres" "sports-centres"})

(with-monad blank-m
  (def rdfstr                    (lift-1 (fn [str] (s str :en))))
  (def replace-comma             (lift-1 (replacer "," ""))  )
  (def trim                      (lift-1 st/trim))
  (def parse-year                (m-chain [trim replace-comma parse-int]))
  (def parse-attendance          (with-monad identity-m (m-chain [(lift-1 (mapper {"" "0"}))
                                                                     (lift-1 (replacer "," ""))
                                                                     trim
                                                                     parse-int])))
  (def convert-month             (m-chain [trim
                                              (lift-1 st/lower-case)
                                              (lift-1 {"january" 1 "jan" 1 "1" 1
                                                       "february" 2 "feb" 2 "2" 2
                                                       "march" 3 "mar" 3 "3" 3
                                                       "april" 4 "apr" 4 "4" 4
                                                       "may" 5 "5" 5
                                                       "june" 6 "jun" 6 "6"  6
                                                       "july" 7 "jul" 7 "7"  7
                                                       "august" 8 "aug" 8 "8" 8
                                                       "september" 9 "sep" 9 "sept" 9 "9"  9
                                                       "october" 10 "oct" 10 "10" 10
                                                       "november" 11 "nov" 11 "11" 11
                                                       "december" 12 "dec" 12 "12" 12
                                                       })]))
  (def convert-year              (m-chain [trim parse-int date-time]))
  (def address-line              (m-chain [trim rdfstr]))
  (def city                      (m-chain [trim rdfstr]))
  (def post-code                 (m-chain [trim rdfstr]))
  (def uriify-pcode              (m-chain [trim
                                              (lift-1 (replacer " " ""))
                                              (lift-1 st/upper-case)
                                              (lift-1 (prefixer "http://data.ordnancesurvey.co.uk/id/postcodeunit/"))]))
  (def url                       (lift-1 #(java.net.URL. %)))
  (def prefix-monthly-attendance (m-chain [(lift-1 date-slug)
                                             (lift-1 (prefixer "http://linked.glasgow.gov.uk/data/glasgow-life-attendances/"))])))

{% endhighlight %}

And now we can understand how everything works:

{% highlight clojure %}
;; rdfstr casts string as an RDF string
cmd-line.prefixers=> (rdfstr "foo")
#<sesame$s$reify__448 foo>

;; replace-comma is quite explicite
cmd-line.prefixers=> (replace-comma "foo,bar")
"foobar"

;; trim removes blanks before and after a string
cmd-line.prefixers=> (trim " foo bar")
"foo bar"

;; parse-year is quite explicite
cmd-line.prefixers=> (parse-year " 2014,")
2014

;; parse-attendance too
cmd-line.prefixers=> (parse-attendance " 0431 ")
431

;; convert-month too
cmd-line.prefixers=> (convert-month " SeptemBer")
9
cmd-line.prefixers=> (convert-month "sep")
9

;; convert-year too
cmd-line.prefixers=> (convert-year 2014)
#<DateTime 2014-01-01T00:00:00.000Z>

;; address-line, city and post-code work the same way
cmd-line.prefixers=> (city "Glasgow")
#<sesame$s$reify__448 Glasgow>

;; uriify-pcode, well, urrify post-codes
cmd-line.prefixers=> (uriify-pcode "G0 431 ")
"http://data.ordnancesurvey.co.uk/id/postcodeunit/G0431"

;; url
cmd-line.prefixers=> (url "http://www.swirrl.com")
#<URL http://www.swirrl.com>


{% endhighlight %}

[Now we can define how to make URI...](911_making_uri.html)
