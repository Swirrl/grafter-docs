---
layout: page
title: 4. Transforming into Graphs
---

**NOTE: This guide covers Grafter 0.6.0**

# Transforming into Graphs

*This is the fourth part of the Grafter Getting Started Guide*

Lets continue with the main file where both the tabular and
graph transformation pipelines are defined.

## Defining graph mappings with graph-fn

Lets look at how to convert a tabular `Dataset` into a graph, through
a `graph-fn` template or mapping.  The following code snippet defines
a function with `graph-fn` that takes each row of data from a
`Dataset`, destructures it by column name and maps it into a linked
data graph with the `graph` function:

{% highlight clojure %}
(def make-graph
  (graph-fn [{:keys [name sex age person-uri gender]}]
            (graph (base-graph "example")
                   [person-uri
                    [rdf:a foaf:Person]
                    [foaf:gender sex]
                    [foaf:age age]
                    [foaf:name (s name)]])))
{% endhighlight %}

A `graph-fn` function has a very similar form to a standard Clojure
anonymous `fn` declaration.  Like `fn` it supports all of Clojure's
[standard binding forms](http://clojure.org/special_forms#binding-forms)
for destructuring.  However unlike `fn`, the function returned by
`graph-fn` takes a `Dataset` table as an argument.  When given a
Dataset `graph-fn` applies its function defition (the body) to each
row of data.  Destructuring each row by the Datasets column names.

First lets have a look at a simpler example:

{% highlight clojure %}
(def mygraphfn
   (graph-fn [row]
             [(->Quad "http://foo" "http://bar" (row "a") "http://graph-name")]))
{% endhighlight %}

Here you can see that `graph-fn` receives a single argument bound to
the name `row` and then explicitly returns a single Linked Data Quad
which is wrapped inside a clojure sequence.  We'll see why we've done
this in a second, but first lets call it at the REPL:

<div class="terminal-wrapper">
  <div class="terminal-inner">test-project.pipeline> (def mygraphfn
                         (graph-fn [row]
                                   [(->Quad "http://foo" "http://bar" (row "a") "http://graph-name")]))

;; => #'test-project.pipeline/mygraphfn
test-project.pipeline=> (def one-row-ds (make-dataset [[1 2 3]]))
#'test-project.pipeline/one-row-ds
test-project.pipeline=> one-row-ds

| a | b | c |
|---+---+---|
| 1 | 2 | 3 |

test-project.pipeline> (mygraphfn one-row-ds)
;; => (#grafter.rdf.protocols.Quad{:s "http://foo", :p "http://bar", :o 1, :c "http://graph-name"})
test-project.pipeline> (mygraphfn (make-dataset [[1 2 3] [4 5 6]]))
;; => (#grafter.rdf.protocols.Quad{:s "http://foo", :p "http://bar", :o 1, :c "http://graph-name"} #grafter.rdf.protocols.Quad{:s "http://foo", :p "http://bar", :o 4, :c "http://graph-name"})</div>
</div>

We can now see what is happening.  First we passed the `mygraphfn`
function a dataset with a single row, and it returned us a `Quad` with
the value from column `a` in its object position.

If we then call it with a two row dataset we can see that it returns
us a second quad, corresponding to the value in this row.

So why did we wrap our `Quad` in a `[]`?  Well that's because each
form inside a graph-fn body is expected to always return a sequence of
Quads.  For example if we redefine `mygraphfn`:

{% highlight clojure %}
(def mygraphfn
  (graph-fn [row]
    [(->Quad "http://foo" "http://bar" (row "a") "http://graph-one")
     (->Quad "http://foo" "http://bar" (row "b") "http://graph-one")]

    [(->Quad "http://foo" "http://bar" (row "c") "http://graph-two")]))
{% endhighlight %}

And then call it at the REPL:

<div class="terminal-wrapper">
  <div class="terminal-inner">test-project.pipeline> test-project.pipeline> (mygraphfn one-row-ds))
;; => (#grafter.rdf.protocols.Quad{:s "http://foo", :p "http://bar", :o 1, :c "http://graph-one"} #grafter.rdf.protocols.Quad{:s "http://foo", :p "http://bar", :o 2, :c "http://graph-one"} #grafter.rdf.protocols.Quad{:s "http://foo", :p "http://bar", :o 3, :c "http://graph-two"})</div>
  </div>

We can see that mygraphfn has returned a single flat sequence of
quads.  So the `graph-fn` macro expects each form in its body to
return a sequence of Quads.  These sequences are then concatenated
together to form a flattened sequence of graph statements.

If we had to write Graph forms like this, things would quickly get
tedious, so we can use the `graph` function in each of these forms to
make things easier.

So lets take a look at `graph`:

<div class="terminal-wrapper">
  <div class="terminal-inner">test-project.pipeline=> (graph "http://graph-uri.com/"
                   #_=>        ["http://subject" ["http://predicate/" 1]])
;; => (#grafter.rdf.protocols.Quad{:s "http://subject", :p "http://property/", :o 2, :c "http://graph-uri.com/"})</div>
</div>

The `graph` function takes a URI for the graph name, and a turtle-like
form of nested vectors `[]` and returns a sequence of statements.
This form associates a single subject with many property, object
pairs.  Because a subject can have many pairs of predicate/objects you
have to provide an extra vector to collect these together.

Lets try `graph` with a more complicated template that shows the
terseness of this format:

<div class="terminal-wrapper">
<div class="terminal-inner">test-project.pipeline=> (graph "http://graph-uri.com/"
                   #_=>        ["http://subject" ["http://property/" 1]
                                                 ["http://property/" 2]])

;; => (#grafter.rdf.protocols.Quad{:s "http://subject", :p "http://property/", :o 1, :c "http://graph-uri.com/"} #grafter.rdf.protocols.Quad{:s "http://subject", :p "http://property/", :o 2, :c "http://graph-uri.com/"})</div>
</div>

As you can see because of the type definitions `graph` forms like this
work with `graph-fn` really nicely.  Allowing you to not just generate
linked data from the source data, but also the graph uri's where you
store it.

## Defining Graph Pipelines with declare-pipeline

So now that we know how to convert tabular data row by row into graph
data we can look at what makes up a graph transformation.

As with tabular transformations graph transformations are just
functions, however we call transformations that return a type of `(Seq
Statement)` grafts.  Transformations wtih this declared return type
are specially supported by Grafter execution environments to for
example output the statements in a chosen RDF serialisation.

Lets take a look at the example defined in the `pipeline.clj` file:

{% highlight clojure %}
(defn convert-persons-data-to-graph
  "Pipeline to convert the tabular persons data sheet into graph data."
  [dataset]
  (->> dataset convert-persons-data make-graph))

(declare-pipeline convert-persons-data-to-graph [Dataset -> Quads]
                  {dataset "The data file to convert into a graph."})
{% endhighlight %}

Again, we see that a graft is just a normal Clojure function, except
it is declared with `declare-pipeline` to have the appropriate return
type of `-> Quads`, which is an alias to represent a sequence of Quads
( `[Quad]`).  Here we receive the same input arguments as the
`convert-persons-data` transformation, so we can both clean the input
file (reusing our previous transformation) and lazily map each row
into an RDF graph.

So long as this function meets the contract of a `graft` (returning
`Quads`) we can continue to compose additional functions on
the end.  For example you may wish to filter out certain quads or
values you don't want to output.

Functions that return a graph like this have special support in the
leiningen plugin for example:

<div class="terminal-wrapper">
  <div class="terminal-inner">$ lein grafter list grafts
Pipeline                                                     Type      Arguments            Description
test-project.pipeline/convert-persons-data-to-graph          graft     data-file            ;; A pipeline that converts the persons data sheet into graph data.</div>
</div>

Grafter supports all main RDF serialisations including turtle
(`.ttl`), n-triples (`.nt`), trig (`.trig`), trix (`.trix`), n-quads
(`.nq`) and RDF XML (`.rdf`).  You can find information on how to use
graft pipelines to generate these serialisations from the command line
in [part two](/getting-started/020-running-pipelines.html).

Grafter also
[provides](http://api.grafter.org/master/grafter.rdf.io.html#var-rdf-serializer)
API [functions](http://api.grafter.org/master/grafter.rdf.html#var-add)
for accessing this functionality, however that is beyond the scope of
this article.

### Running a graft at the REPL

You should now see the Clojure REPL started in your projects pipeline namespace.  First lets try running the graft `convert-persons-data-to-graph`:

<div class="terminal-wrapper">
  <div class="terminal-inner">
test-project.pipeline=&gt; (convert-persons-data-to-graph "./data/example-data.csv")

(#grafter.rdf.protocols.Quad{:s "http://my-domain.com/id/Alice", :p "http://www.w3.org/1999/02/22-rdf-syntax-ns#type", :o "http://xmlns.com/foaf/0.1/Person", :c "http://my-domain.com/graph/example"} #grafter.rdf.protocols.Quad{:s "http://my-domain.com/id/Alice", :p "http://xmlns.com/foaf/0.1/gender", :o #<io$s$reify__9455 female>, :c "http://my-domain.com/graph/example"} #grafter.rdf.protocols.Quad{:s "http://my-domain.com/id/Alice", :p "http://xmlns.com/foaf/0.1/age", :o 34, :c "http://my-domain.com/graph/example"} #grafter.rdf.protocols.Quad{:s "http://my-domain.com/id/Alice", :p "http://xmlns.com/foaf/0.1/name", :o #<io$s$reify__9455 Alice>, :c "http://my-domain.com/graph/example"} #grafter.rdf.protocols.Quad{:s "http://my-domain.com/id/Bob", :p "http://www.w3.org/1999/02/22-rdf-syntax-ns#type", :o "http://xmlns.com/foaf/0.1/Person", :c "http://my-domain.com/graph/example"} #grafter.rdf.protocols.Quad{:s "http://my-domain.com/id/Bob", :p "http://xmlns.com/foaf/0.1/gender", :o #<io$s$reify__9455 male>, :c "http://my-domain.com/graph/example"} #grafter.rdf.protocols.Quad{:s "http://my-domain.com/id/Bob", :p "http://xmlns.com/foaf/0.1/age", :o 63, :c "http://my-domain.com/graph/example"} #grafter.rdf.protocols.Quad{:s "http://my-domain.com/id/Bob", :p "http://xmlns.com/foaf/0.1/name", :o #<io$s$reify__9455 Bob>, :c "http://my-domain.com/graph/example"})
</div>
</div>

Here you can see how the graft transformed the file path into a list of `quads` ready to be exported in a RDF graph by Grafter. The return value of the complete graft should always be a list of `quads` otherwise you'll likely get an error if you run the pipeline in other contexts.

As we can see a `quad` is quite similar to a `RDF triple`. It's a Clojure's `hash-map` which contains three keys `:s`, `:p` and `o`, corresponding to a triple `subject`, `predicate` and `object` and one key `c` corresponding to the graph.

Lets investigate the grafts steps one by one so we can see what's going on. It starts with a pipe. If you havn't yet, you can read the [understanding pipes section](./030-understanding-pipes-html).

<div class="terminal-wrapper">
  <div class="terminal-inner">
test-project.pipeline=&gt; (convert-persons-data "./data/example-data.csv")

| :name |   :sex | :age |                   :person-uri |
|-------+--------+------+-------------------------------|
| Alice | female |   34 | http://my-domain.com/id/Alice |
|   Bob |   male |   63 |   http://my-domain.com/id/Bob |

</div>
</div>

Next up enter `(->> (convert-persons-data "./data/example-data.csv") make-graph)`.

<div class="terminal-wrapper">
  <div class="terminal-inner">
test-project.pipeline=&gt; (->> (convert-persons-data "./data/example-data.csv")
                            make-graph)

(#grafter.rdf.protocols.Quad{:s "http://my-domain.com/id/Alice", :p "http://www.w3.org/1999/02/22-rdf-syntax-ns#type", :o "http://xmlns.com/foaf/0.1/Person", :c "http://my-domain.com/graph/example"} #grafter.rdf.protocols.Quad{:s "http://my-domain.com/id/Alice", :p "http://xmlns.com/foaf/0.1/gender", :o #<io$s$reify__9455 female>, :c "http://my-domain.com/graph/example"} #grafter.rdf.protocols.Quad{:s "http://my-domain.com/id/Alice", :p "http://xmlns.com/foaf/0.1/age", :o 34, :c "http://my-domain.com/graph/example"} #grafter.rdf.protocols.Quad{:s "http://my-domain.com/id/Alice", :p "http://xmlns.com/foaf/0.1/name", :o #<io$s$reify__9455 Alice>, :c "http://my-domain.com/graph/example"} #grafter.rdf.protocols.Quad{:s "http://my-domain.com/id/Bob", :p "http://www.w3.org/1999/02/22-rdf-syntax-ns#type", :o "http://xmlns.com/foaf/0.1/Person", :c "http://my-domain.com/graph/example"} #grafter.rdf.protocols.Quad{:s "http://my-domain.com/id/Bob", :p "http://xmlns.com/foaf/0.1/gender", :o #<io$s$reify__9455 male>, :c "http://my-domain.com/graph/example"} #grafter.rdf.protocols.Quad{:s "http://my-domain.com/id/Bob", :p "http://xmlns.com/foaf/0.1/age", :o 63, :c "http://my-domain.com/graph/example"} #grafter.rdf.protocols.Quad{:s "http://my-domain.com/id/Bob", :p "http://xmlns.com/foaf/0.1/name", :o #<io$s$reify__9455 Bob>, :c "http://my-domain.com/graph/example"})

test-project.pipeline=&gt;
</div>
</div>

You'll notice that `make-graph` takes a `Dataset`, destructures its header, and returns the `quads` corresponding. We'll talk more about how to define `make-graph` functions in another section.

### Slightly more complex example

When you work with data you often have to deal with missing data. *Lets play a little bit more with the REPL and Grafter to see how to do this!*

First lets create a new `Dataset` with a missing data:

<div class="terminal-wrapper">
  <div class="terminal-inner">
test-project.pipeline=&gt; (def new-data-example (make-dataset [["name" "sex" "age"] ["Alice" "f" "34"] ["Bob" nil "63"]]))
#'test-project.pipeline/new-data-example

test-project.pipeline=&gt; new-data-example

|     a |   b |   c |
|-------+-----+-----|
|  name | sex | age |
| Alice |   f |  34 |
|   Bob |     |  63 |</div>
</div>

We can test our function just for fun:

<div class="terminal-wrapper">
  <div class="terminal-inner">

test-project.pipeline=&gt; (convert-persons-data new-data-example)

| :name |   :sex | :age |                   :person-uri |
|-------+--------+------+-------------------------------|
| Alice | female |   34 | http://my-domain.com/id/Alice |
|   Bob |        |   63 |   http://my-domain.com/id/Bob |</div>
</div>

And if we use our `graft`:

<div class="terminal-wrapper">
  <div class="terminal-inner">

test-project.pipeline=&gt; (convert-persons-data-to-graph new-data-example)

(#grafter.rdf.protocols.Quad{:s "http://my-domain.com/id/Alice", :p "http://www.w3.org/1999/02/22-rdf-syntax-ns#type", :o "http://xmlns.com/foaf/0.1/Person", :c "http://my-domain.com/graph/example"} #grafter.rdf.protocols.Quad{:s "http://my-domain.com/id/Alice", :p "http://xmlns.com/foaf/0.1/gender", :o #<io$s$reify__9455 female>, :c "http://my-domain.com/graph/example"} #grafter.rdf.protocols.Quad{:s "http://my-domain.com/id/Alice", :p "http://xmlns.com/foaf/0.1/age", :o 34, :c "http://my-domain.com/graph/example"} #grafter.rdf.protocols.Quad{:s "http://my-domain.com/id/Alice", :p "http://xmlns.com/foaf/0.1/name", :o #<io$s$reify__9455 Alice>, :c "http://my-domain.com/graph/example"} #grafter.rdf.protocols.Quad{:s "http://my-domain.com/id/Bob", :p "http://www.w3.org/1999/02/22-rdf-syntax-ns#type", :o "http://xmlns.com/foaf/0.1/Person", :c "http://my-domain.com/graph/example"} #grafter.rdf.protocols.Quad{:s "http://my-domain.com/id/Bob", :p "http://xmlns.com/foaf/0.1/gender", :o nil, :c "http://my-domain.com/graph/example"} #grafter.rdf.protocols.Quad{:s "http://my-domain.com/id/Bob", :p "http://xmlns.com/foaf/0.1/age", :o 63, :c "http://my-domain.com/graph/example"} #grafter.rdf.protocols.Quad{:s "http://my-domain.com/id/Bob", :p "http://xmlns.com/foaf/0.1/name", :o #<io$s$reify__9455 Bob>, :c "http://my-domain.com/graph/example"})</div>
</div>

We can notice a `quad` with a missing `object`: `#grafter.rdf.protocols.Quad{:s "http://my-domain.com/id/Bob", :p "http://xmlns.com/foaf/0.1/gender", :o nil, :c "http://my-domain.com/graph/example"}`.

To fix this problem we can `remove` triples with a nil `object`. First lets define a pretty simple filter:

<div class="terminal-wrapper">
  <div class="terminal-inner">
test-project.pipeline=&gt; (defn missing-data-filter [triples]
                               (remove #(nil? (pr/object %)) triples))
#'test-project.pipeline/missing-data-filter</div>
</div>

And then we create the new `graft`:

<div class="terminal-wrapper">
  <div class="terminal-inner">

test-project.pipeline=&gt; (defn convert-persons-data-to-graph-with-filter [dataset]
                              (->> dataset convert-persons-data make-graph missing-data-filter))
#'test-project.pipeline/convert-persons-data-to-graph-with-filter</div>
</div>

And test it:

<div class="terminal-wrapper">
  <div class="terminal-inner">
test-project.pipeline=&gt; (convert-persons-data-to-graph-with-filter new-data-example)
(#grafter.rdf.protocols.Quad{:s "http://my-domain.com/id/Alice", :p "http://www.w3.org/1999/02/22-rdf-syntax-ns#type", :o "http://xmlns.com/foaf/0.1/Person", :c "http://my-domain.com/graph/example"} #grafter.rdf.protocols.Quad{:s "http://my-domain.com/id/Alice", :p "http://xmlns.com/foaf/0.1/gender", :o #<io$s$reify__9455 female>, :c "http://my-domain.com/graph/example"} #grafter.rdf.protocols.Quad{:s "http://my-domain.com/id/Alice", :p "http://xmlns.com/foaf/0.1/age", :o 34, :c "http://my-domain.com/graph/example"} #grafter.rdf.protocols.Quad{:s "http://my-domain.com/id/Alice", :p "http://xmlns.com/foaf/0.1/name", :o #<io$s$reify__9455 Alice>, :c "http://my-domain.com/graph/example"} #grafter.rdf.protocols.Quad{:s "http://my-domain.com/id/Bob", :p "http://www.w3.org/1999/02/22-rdf-syntax-ns#type", :o "http://xmlns.com/foaf/0.1/Person", :c "http://my-domain.com/graph/example"} #grafter.rdf.protocols.Quad{:s "http://my-domain.com/id/Bob", :p "http://xmlns.com/foaf/0.1/age", :o 63, :c "http://my-domain.com/graph/example"} #grafter.rdf.protocols.Quad{:s "http://my-domain.com/id/Bob", :p "http://xmlns.com/foaf/0.1/name", :o #<io$s$reify__9455 Bob>, :c "http://my-domain.com/graph/example"})</div>
</div>

And there isn't any `nil` object anymore!

Congratulations, you've just reached the end of the Grafter quick
start guide.  We hope you've enjoyed it and found it informative.  We
have only touched on a fraction of what is possible with Grafter, so
please dig deeper and feel free to contribute to ask questions and
contribute in the Grafter & Clojure community.
