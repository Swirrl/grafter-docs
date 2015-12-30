---
layout: page
title: 3. Transforming into Tables
---

**NOTE: This guide covers Grafter 0.6.0**

# Transforming into Tables

*This is the third part of the Grafter Getting Started Guide.*

Let's take a look at where both the tabular and graph transformations are defined.

Let's start by looking at the tabular definition:

{% highlight clojure %}
(defn convert-persons-data
  "Pipeline to convert tabular persons data into a different tabular format."
  [data-file]
  (-> (read-dataset data-file)
      (drop-rows 1)
      (make-dataset [:name :sex :age])
      (derive-column :person-uri [:name] base-id)
      (mapc {:age ->integer
             :sex {"f" (s "female")
                   "m" (s "male")}})))

(declare-pipeline convert-persons-data [Dataset -> Dataset]
                  {data-file "A data file"})
{% endhighlight %}

The first thing to notice about grafter transformations is that they
are just normal Clojure functions.  However you'll notice that below
the function call there is a `declare-pipeline` call which advertises
that function to the plugin and other Grafter services, whilst
providing extra information on the pipelines expected types along with
doc strings for each argument.

The type declaration `Dataset -> Dataset` specifies that this
transformation converts a `Dataset` (table) into another tabular
`Dataset`.  (Clojure experts may notice that the type forms in 0.6.0
are syntactically the same as in core.typed, however we expect to
change the syntax of these forms in 0.7.0 to follow that of
prismatic/schema)

These annotations support extra tooling, such as the leiningen plugin
which can be run the at the command line with `lein grafter list`.
The plugin will then display appropriate information about the
pipeline and use the type declarations to coerce its command line
arguments into the expected types for the pipeline.

`declare-pipeline` allows this tooling to distinguish top level
transformation functions from ordinary functions.

## Working at the Clojure REPL

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

### Running a pipeline

You should now see the Clojure REPL started in your projects pipeline
namespace.  First let's try running the pipe `convert-persons-data
which we ran via the plugin before:

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

Here you can see how as before the pipe translated the file path into
a transformed `Dataset`.  The return value of the complete pipe should
always be a `Dataset`, otherwise you'll likely get an error if you run
the pipeline in other contexts.

### Reading a dataset

The datasets used by Grafter are interoperable with Incanter Dataset records and
share the same internal representation.  Like all Clojure records they can be
thought of as hash maps.  They contain two fields named `:column-names` and
`:rows`.  Columns are stored as an ordered vector of `:column-names`, whilst
`:rows` contains a sequence of maps where the keys are from `:column-names` and
the values are the contentes of that specific cell.

{% highlight clojure %}
;; A pretty-printed Dataset

{:column-names [:name :age :person-uri],
 :rows
 ({:age 34,
   :person-uri "http://my-domain.com/id/Alice",
   :name "Alice"}
  {:age 63,
   :person-uri "http://my-domain.com/id/Bob",
   :name "Bob"})}
{% endhighlight %}

Let's investigate the pipes steps one by one so we can see what's going
on.  First let's try using `read-dataset` to open our example file:

<div class="terminal-wrapper">
<div class="terminal-inner">test-project.pipeline=&gt; (read-dataset "./data/example-data.csv")

|     a |   b |   c |
|-------+-----+-----|
|  name | sex | age |
| Alice |   f |  34 |
|   Bob |   m |  63 |
</div>
</div>

You'll notice that `read-dataset` has coerced the `String` into a file
path, and has seen the file extension which likely indicates a CSV
file, causing it to open it as a `Dataset`.

Notice though that Grafter can't assume that the first row of the data
are actually the column headings, so it has initialised the dataset
with the three column names `a`, `b` and `c`.

## Composing Pipelines

Typically when building Grafter transformations for flexibility we
split the pipeline into two parts.  The tabular transformation
(typically declared as `Dataset -> Dataset`), and the graph
transformation (typically declared `Dataset -> Quads`).
For short we commonly refer to these as `pipe`s and `graft`s.

Spliting pipelines in this way helps maximise code reuse and aids in
debugging as you can easily see what transformed table was generated
before being converted into an RDF graph.

We will now take a look at building the above pipe step by step.

### Drop the header with drop-rows

Next up enter `(-> (read-dataset "./data/example-data.csv") (drop-rows
1))`.

<div class="terminal-wrapper">
<div class="terminal-inner">test-project.pipeline=> (-> (read-dataset "./data/example-data.csv")
                   #_=>     (drop-rows 1))

|     a | b |  c |
|-------+---+----|
| Alice | f | 34 |
|   Bob | m | 63 |</div>
</div>

*Note that the `#_=>` shouldn't be entered as its inserted by the REPL
to keep your code aligned.*

You'll see that this removes the header row from the data.

### Name the header row with make-dataset

`read-dataset` already returns us a valid dataset, however we can
choose to set our own headers by issuing another call to
`make-dataset`.  Its worth noting that because of Clojure's immutable
datastructures, technically every step in a pipeline creates a new
dataset.  Thanks to Clojure's persistent datastructures this is an
efficient operation.

So, let's make those headers:

<div class="terminal-wrapper">
<div class="terminal-inner">test-project.pipeline=> (-> (read-dataset "./data/example-data.csv")
                   #_=>     (drop-rows 1)
                   #_=>     (make-dataset [:name :sex :age]))

| :name | :sex | :gender |
|-------+------+---------|
| Alice |    f |      34 |
|   Bob |    m |      63 |</div>
</div>

`make-dataset` is interesting because it is main purpose is to
construct new datasets from Clojure sequences, e.g. you can convert a
sequence of sequences into a dataset like so:

<div class="terminal-wrapper">
<div class="terminal-inner">test-project.pipeline=> (make-dataset [[1 2 3] [4 5 6]])

| a | b | c |
|---+---+---|
| 1 | 2 | 3 |
| 4 | 5 | 6 |</div>
</div>

And if you want to explicitly set the column names you can by
supplying a sequence of column names as its final argument:

<div class="terminal-wrapper">
<div class="terminal-inner">test-project.pipeline=> (make-dataset [[1 2 3] [4 5 6]] [:first :second :third])

| :first | :second | :third |
|--------+---------+--------|
|      1 |       2 |      3 |
|      4 |       5 |      6 |</div>
</div>

The approach taken so far in this pipe has been to remove the header
row and explicitly set it to known values.  You can also supply a
function to `make-dataset` to move the first row from the data into
your headings for you.  Grafter defines a function for this job which
is `move-first-row-to-header` and it can be used like this:

<div class="terminal-wrapper">
<div class="terminal-inner">test-project.pipeline=> (-> (make-dataset [["one" "two" "three"] [1 2 3] [4 5 6]])
                   #_=>     (make-dataset move-first-row-to-header))

| one | two | three |
|-----+-----+-------|
|   1 |   2 |     3 |
|   4 |   5 |     6 |</div>
</div>

Column names can be used by you, the Grafter developer, to get a
handle on the data, and identify each cell as grafter is iterating
through all the rows.  Consequently if you derive your column names
from the data you need to be sure that they're not going to change
when the data is updated.

Obviously some column names are variable and expected to change, for
example you might have a spreadsheet of years.  There are ways to
handle situations like this via functions like `melt`, or by
addressing columns with ranges and sequences.  This is a more advanced
topic for another day.

### Derive a new column with derive-column

Next let's use Grafter's `derive-column` to derive a new URI for each
person in the source data by concatenating a URI prefix with each
persons name.

`derive-column` takes an input dataset, a new column name and a vector
of source columns followed by a function to perform the
transformation.  `derive-column` will then execute the function on
each row, selecting the source columns and applying the values from
those cells to the supplied function.  The resulting value is then
added to the dataset in the new column.  Essentially `derive-column`
is `apply` for `Dataset`s, where the return value is stored in a new
column.

First let's have a play with it at the REPL:

<div class="terminal-wrapper">
<div class="terminal-inner">test-project.pipeline=> (-> (make-dataset [[1 2 3] [4 5 6]] [:a :b :c])
                   #_=>     (derive-column :d [:b :c] +))

| :a | :b | :c | :d |
|----+----+----+----|
|  1 |  2 |  3 |  5 |
|  4 |  5 |  6 | 11 |</div>
</div>

`derive-column` can also accept zero indexed positional identifiers
for source columns.  As `derive-column` always adds the new columns at
the end of the dataset we still need to provide a column name:

<div class="terminal-wrapper">
<div class="terminal-inner">test-project.pipeline=> (-> (make-dataset [[1 2 3] [4 5 6]] [:a :b :c])
                   #_=>     (derive-column :d [1 2] +))

| :a | :b | :c | :d |
|----+----+----+----|
|  1 |  2 |  3 |  5 |
|  4 |  5 |  6 | 11 |</div>
</div>

So let's try and generate a uri, there is a function defined in the
template for building resource-id URI's already in the
`test-project.prefix` namespace.  Let's try and call it:

<div class="terminal-wrapper">
<div class="terminal-inner">test-project.pipeline=> (base-id "foo")
"http://my-domain.com/id/foo"
</div>
</div>

We can see that this function concatenates its argument to a base
prefix for us (if you want a different prefix you can redifine the
function definition in `test-project.prefix`).

So let's try adding using this with our pipeline so far and
`derive-column`:

<div class="terminal-wrapper">
<div class="terminal-inner">test-project.pipeline=> (-> (read-dataset "./data/example-data.csv")
                   #_=>       (drop-rows 1)
                   #_=>       (make-dataset [:name :sex :age])
                   #_=>       (derive-column :person-uri [:name] base-id))

| :name | :sex | :age |                   :person-uri |
|-------+------+------+-------------------------------|
| Alice |    f |   34 | http://my-domain.com/id/Alice |
|   Bob |    m |   63 |   http://my-domain.com/id/Bob |</div>
</div>

You can see how this has added a new column called `:person-uri` to
the source data with URI's built from the persons name.

### Change values in a column with mapc

`derive-column` is incredibly useful, however sometimes you just want
to apply a function transformation to a single cell, and put the value
back in that cell.  `mapc` assumes that each function it is given is
operating on a single source cell, so you have to supply it with an
arity-1 (i.e. a single argument) function.

However `mapc` lets you operate on multiple columns at the same time,
so you simply need to associate functions with the column names that
they're to operate on.  To do this we can use a hash-map:

For example the following code defines an example dataset and then
increments all the values in column `:a` with Clojure's `inc` function
and decrements all the values in column `:b` with `dec`:

<div class="terminal-wrapper">
<div class="terminal-inner">test-project.pipeline=> (-> (make-dataset [[1 2 3] [4 5 6]])
                   #_=>     (mapc {:a inc :b dec}))

| a | b | c |
|---+---+---|
| 2 | 1 | 3 |
| 5 | 4 | 6 |</div>
</div>

So back in our example pipe, we want make sure that we cast all the
ages to integers (as we're reading from CSV everything is implicitly
read in as a `String`).

The template has defined an example function to conver the Strings to
integers in the namespace `test-project.transform`:

{% highlight clojure %}
(defn ->integer
  "An example transformation function that converts a string to an integer"
  [s]
  (Integer/parseInt s))
{% endhighlight clojure %}

This function simply delegates to a Java method to convert a String
into an Integer.

While we're here we want to convert the codes `f` and `m` into
something more descriptive; like the string values `male` and
`female`.  We can do this by making use of the fact that in Clojure
hash-maps are functions from their keys to their values e.g.

<div class="terminal-wrapper">
<div class="terminal-inner">test-project.pipeline=> test-project.pipeline=> ({"f" "female" "m" "male"} "f")
"female"
test-project.pipeline=> ({"f" "female" "m" "male"} "m")
"male"
test-project.pipeline=> ({"f" "female" "m" "male"} "z")
nil</div>
</div>

So we can try doing both of these things to our pipeline at the same
time like this:

{% highlight clojure %}
(-> (read-dataset data-file)
      (drop-rows 1)
      (make-dataset [:name :sex :age])
      (derive-column :person-uri [:name] base-id)
      (mapc {:age ->integer
             :sex {"f" (s "female")
                   "m" (s "male")}}))
{% endhighlight clojure %}

*NOTE because we're later going to convert this data into linked data
 with a `graft` step, we also make use of the `s` function which
 converts java `String` objects into Linked Data strings.*


## Exposing Grafter Services

An important architectural principle of Grafter is that it makes a
clean separation between a pipelines transformation responsibilities,
and the execution environment, or service within which it runs.

There are currently several different execution environments available
for Grafter.  These are:

- The leiningen plugin (Open Source)
- Grafter Server (Proprietary - but planned to be open sourced)
- The Graftwerk service wrapper (Proprietary - planned to be opened)

These services, like the leiningen plugin, use information exposed
through the `declare-pipeline` directive to build a web based forms
and RESTful services for every declared grafter pipeline.

For example these servers can use the type declarations to build a web
based form for your pipeline, converting and coercing parameters and
files from a HTTP request into the objects your pipeline is expecting.

The pipeline never needs to say where to put the output data, as the
output destination, whether it's a file a SPARQL endpoint or something
else is the responsibility of the execution environment in which the
pipeline is run.
