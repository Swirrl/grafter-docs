---
layout: page
title: 4. Understanding Grafts
---

# Understanding Grafts

*This is the fourth part of the Grafter Getting Started Guide*

Lets continue with the main file where both the `pipe`and `graft` transformation pipelines are defined.

Lets focus on the `defgraft` definition:

{% highlight clojure %}

(defgraft convert-persons-data-to-graph
  "Pipeline to convert the tabular persons data sheet into graph data."
  convert-persons-data make-graph)

{% endhighlight %}

While `defpipe` forms are syntactically identical to Clojure's `defn` form, `defgraft` forms are syntactically identical to Clojure's `def` form. Indeed, `defgraft` creates a var with the **composition** of the functions given as arguments and advertises that var to the plugin and other Grafter services.

For example if you run the command `lein grafter list grafts` the plugin
will search the projects classpath for any clj files with valid
defgraft definitions, and list them.

`defgraft` is necessary as it allows Grafter to distinguish `grafts` from ordinary functions.

## What is a graft?

A graft is any function defined with Grafters `defgraft`, that can convert 0 or more `datasettable` arguments into an `RDF` graph output.

Dataset's are how Grafter represents tabular data, and `datasettables`
are said to be any type which Grafter can coerce into a `Dataset`.  This
includes for example Strings representing file paths or URLs,
`java.io.File` objects, `java.net.URI` and `URL` objects, along with
`InputStream`s and `Reader`s, and various others.  Importantly
`Dataset`s are also `datasettable`.

Grafter supports all main RDF serialisations including turtle
(`.ttl`), n-triples (`.nt`), trig (`.trig`), trix (`.trix`), n-quads
(`.nq`) and RDF XML (`.rdf`).  Again the desired format is infered by
grafter from the file extension.

Typically a `defgraft` definition will as its first action use a Grafters `pipe`, apply a `make-graph` function, creating `RDF` triples from the `Dataset`, and possibly, apply one or more filters to get rid of triples with missing data.

Its important to note that graft's are functions from `datasettable* -> graph` and that so long as a graft meets this contract it can be used by the Grafter plugin and other Grafter services.

## Running Transformations at the Clojure REPL

### First example

The best way to understand Grafter is to play with it at the Clojure
REPL.  If you already have a Clojure environment with good editor
integration then you should start a REPL there, otherwise you can
start a basic REPL by running:

<div class="terminal-wrapper">
  <div class="terminal-inner">$ lein repl
nREPL server started on port 63955 on host 127.0.0.1 - nrepl://127.0.0.1:63955
REPL-y 0.3.5, nREPL 0.2.6
Clojure 1.6.0
Java HotSpot(TM) 64-Bit Server VM 1.8.0_25-b17
    Docs: (doc function-name-here)
          (find-doc "part-of-name-here")
  Source: (source function-name-here)
 Javadoc: (javadoc java-object-or-class-here)
    Exit: Control+D or (exit) or (quit)
 Results: Stored in vars *1, *2, *3, an exception in *e

test-project.pipeline=&gt;</div>
</div>

You should now see the Clojure REPL started in your projects pipeline namespace.  First lets try running the graft `convert-persons-data-to-graph`:

<div class="terminal-wrapper">
  <div class="terminal-inner">
test-project.pipeline=&gt; (convert-persons-data-to-graph "./data/example-data.csv")

(#grafter.rdf.protocols.Quad{:s "http://my-domain.com/id/Alice", :p "http://www.w3.org/1999/02/22-rdf-syntax-ns#type", :o "http://xmlns.com/foaf/0.1/Person", :c "http://my-domain.com/graph/example"} #grafter.rdf.protocols.Quad{:s "http://my-domain.com/id/Alice", :p "http://xmlns.com/foaf/0.1/gender", :o #<io$s$reify__9455 female>, :c "http://my-domain.com/graph/example"} #grafter.rdf.protocols.Quad{:s "http://my-domain.com/id/Alice", :p "http://xmlns.com/foaf/0.1/age", :o 34, :c "http://my-domain.com/graph/example"} #grafter.rdf.protocols.Quad{:s "http://my-domain.com/id/Alice", :p "http://xmlns.com/foaf/0.1/name", :o #<io$s$reify__9455 Alice>, :c "http://my-domain.com/graph/example"} #grafter.rdf.protocols.Quad{:s "http://my-domain.com/id/Bob", :p "http://www.w3.org/1999/02/22-rdf-syntax-ns#type", :o "http://xmlns.com/foaf/0.1/Person", :c "http://my-domain.com/graph/example"} #grafter.rdf.protocols.Quad{:s "http://my-domain.com/id/Bob", :p "http://xmlns.com/foaf/0.1/gender", :o #<io$s$reify__9455 male>, :c "http://my-domain.com/graph/example"} #grafter.rdf.protocols.Quad{:s "http://my-domain.com/id/Bob", :p "http://xmlns.com/foaf/0.1/age", :o 63, :c "http://my-domain.com/graph/example"} #grafter.rdf.protocols.Quad{:s "http://my-domain.com/id/Bob", :p "http://xmlns.com/foaf/0.1/name", :o #<io$s$reify__9455 Bob>, :c "http://my-domain.com/graph/example"})

test-project.pipeline=&gt;
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

test-project.pipeline=&gt;
</div>
</div>

Next up enter `(-> (convert-persons-data "./data/example-data.csv") make-graph)`.

<div class="terminal-wrapper">
  <div class="terminal-inner">
test-project.pipeline=&gt; (-> (convert-persons-data "./data/example-data.csv") 
                            make-graph)

(#grafter.rdf.protocols.Quad{:s "http://my-domain.com/id/Alice", :p "http://www.w3.org/1999/02/22-rdf-syntax-ns#type", :o "http://xmlns.com/foaf/0.1/Person", :c "http://my-domain.com/graph/example"} #grafter.rdf.protocols.Quad{:s "http://my-domain.com/id/Alice", :p "http://xmlns.com/foaf/0.1/gender", :o #<io$s$reify__9455 female>, :c "http://my-domain.com/graph/example"} #grafter.rdf.protocols.Quad{:s "http://my-domain.com/id/Alice", :p "http://xmlns.com/foaf/0.1/age", :o 34, :c "http://my-domain.com/graph/example"} #grafter.rdf.protocols.Quad{:s "http://my-domain.com/id/Alice", :p "http://xmlns.com/foaf/0.1/name", :o #<io$s$reify__9455 Alice>, :c "http://my-domain.com/graph/example"} #grafter.rdf.protocols.Quad{:s "http://my-domain.com/id/Bob", :p "http://www.w3.org/1999/02/22-rdf-syntax-ns#type", :o "http://xmlns.com/foaf/0.1/Person", :c "http://my-domain.com/graph/example"} #grafter.rdf.protocols.Quad{:s "http://my-domain.com/id/Bob", :p "http://xmlns.com/foaf/0.1/gender", :o #<io$s$reify__9455 male>, :c "http://my-domain.com/graph/example"} #grafter.rdf.protocols.Quad{:s "http://my-domain.com/id/Bob", :p "http://xmlns.com/foaf/0.1/age", :o 63, :c "http://my-domain.com/graph/example"} #grafter.rdf.protocols.Quad{:s "http://my-domain.com/id/Bob", :p "http://xmlns.com/foaf/0.1/name", :o #<io$s$reify__9455 Bob>, :c "http://my-domain.com/graph/example"})

test-project.pipeline=&gt;
</div>
</div>

You'll notice that `make-graph` takes a `Dataset`, destructures its header, and returns the `quads` corresponding. We'll talk more about how to define `make-graph` functions in another section.

### Slightly more complex example

When you work with data you often have to deal with missing data. The `defgraft` form allows us to filter those missing data: **lets play a little bit more with the Clojure's REPL and with Grafter!**

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
|   Bob |     |  63 |


test-project.pipeline=&gt;
</div>
</div>

We can test our `pipe` just for fun:

<div class="terminal-wrapper">
  <div class="terminal-inner">

test-project.pipeline=&gt; (convert-persons-data new-data-example)

| :name |   :sex | :age |                   :person-uri |
|-------+--------+------+-------------------------------|
| Alice | female |   34 | http://my-domain.com/id/Alice |
|   Bob |        |   63 |   http://my-domain.com/id/Bob |

test-project.pipeline=&gt;
</div>
</div>

And if we use our `graft`:

<div class="terminal-wrapper">
  <div class="terminal-inner">

test-project.pipeline=&gt; (convert-persons-data-to-graph new-data-example)

(#grafter.rdf.protocols.Quad{:s "http://my-domain.com/id/Alice", :p "http://www.w3.org/1999/02/22-rdf-syntax-ns#type", :o "http://xmlns.com/foaf/0.1/Person", :c "http://my-domain.com/graph/example"} #grafter.rdf.protocols.Quad{:s "http://my-domain.com/id/Alice", :p "http://xmlns.com/foaf/0.1/gender", :o #<io$s$reify__9455 female>, :c "http://my-domain.com/graph/example"} #grafter.rdf.protocols.Quad{:s "http://my-domain.com/id/Alice", :p "http://xmlns.com/foaf/0.1/age", :o 34, :c "http://my-domain.com/graph/example"} #grafter.rdf.protocols.Quad{:s "http://my-domain.com/id/Alice", :p "http://xmlns.com/foaf/0.1/name", :o #<io$s$reify__9455 Alice>, :c "http://my-domain.com/graph/example"} #grafter.rdf.protocols.Quad{:s "http://my-domain.com/id/Bob", :p "http://www.w3.org/1999/02/22-rdf-syntax-ns#type", :o "http://xmlns.com/foaf/0.1/Person", :c "http://my-domain.com/graph/example"} #grafter.rdf.protocols.Quad{:s "http://my-domain.com/id/Bob", :p "http://xmlns.com/foaf/0.1/gender", :o nil, :c "http://my-domain.com/graph/example"} #grafter.rdf.protocols.Quad{:s "http://my-domain.com/id/Bob", :p "http://xmlns.com/foaf/0.1/age", :o 63, :c "http://my-domain.com/graph/example"} #grafter.rdf.protocols.Quad{:s "http://my-domain.com/id/Bob", :p "http://xmlns.com/foaf/0.1/name", :o #<io$s$reify__9455 Bob>, :c "http://my-domain.com/graph/example"})

test-project.pipeline=&gt;
</div>
</div>

We can notice a `quad` with a missing `object`: `#grafter.rdf.protocols.Quad{:s "http://my-domain.com/id/Bob", :p "http://xmlns.com/foaf/0.1/gender", :o nil, :c "http://my-domain.com/graph/example"}`.

Since the `defgraft` form is based on `composition`, we can add a filter to remove triples with a nil `object`. First lets define a pretty simple filter:


<div class="terminal-wrapper">
  <div class="terminal-inner">

test-project.pipeline=&gt; (defn missing-data-filter [triples]
                          (filter #(not (nil? (pr/object %))) triples))
#'test-project.pipeline/missing-data-filter

test-project.pipeline=&gt; 

</div>
</div>

And then we create the new `graft`:

<div class="terminal-wrapper">
  <div class="terminal-inner">

test-project.pipeline=&gt; (defgraft convert-persons-data-to-graph-with-filter
                         convert-persons-data make-graph missing-data-filter)
#'test-project.pipeline/convert-persons-data-to-graph-with-filter

test-project.pipeline=&gt;
</div>
</div>

And test it:

<div class="terminal-wrapper">
  <div class="terminal-inner">

test-project.pipeline=&gt; (convert-persons-data-to-graph-with-filter new-data-example)
(#grafter.rdf.protocols.Quad{:s "http://my-domain.com/id/Alice", :p "http://www.w3.org/1999/02/22-rdf-syntax-ns#type", :o "http://xmlns.com/foaf/0.1/Person", :c "http://my-domain.com/graph/example"} #grafter.rdf.protocols.Quad{:s "http://my-domain.com/id/Alice", :p "http://xmlns.com/foaf/0.1/gender", :o #<io$s$reify__9455 female>, :c "http://my-domain.com/graph/example"} #grafter.rdf.protocols.Quad{:s "http://my-domain.com/id/Alice", :p "http://xmlns.com/foaf/0.1/age", :o 34, :c "http://my-domain.com/graph/example"} #grafter.rdf.protocols.Quad{:s "http://my-domain.com/id/Alice", :p "http://xmlns.com/foaf/0.1/name", :o #<io$s$reify__9455 Alice>, :c "http://my-domain.com/graph/example"} #grafter.rdf.protocols.Quad{:s "http://my-domain.com/id/Bob", :p "http://www.w3.org/1999/02/22-rdf-syntax-ns#type", :o "http://xmlns.com/foaf/0.1/Person", :c "http://my-domain.com/graph/example"} #grafter.rdf.protocols.Quad{:s "http://my-domain.com/id/Bob", :p "http://xmlns.com/foaf/0.1/age", :o 63, :c "http://my-domain.com/graph/example"} #grafter.rdf.protocols.Quad{:s "http://my-domain.com/id/Bob", :p "http://xmlns.com/foaf/0.1/name", :o #<io$s$reify__9455 Bob>, :c "http://my-domain.com/graph/example"})

test-project.pipeline=&gt;
</div>
</div>

And there isn't any `nil` object anymore!
