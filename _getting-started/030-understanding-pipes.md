---
layout: page
title: 3. Understanding Pipes
---

# Understanding Pipes

*This is the third part of the Grafter Getting Started Guide.*

Lets take a look at the main file where both the `pipe` and `graft`
transformation pipelines are defined.

Lets start by looking at the `defpipe` definition:

{% highlight clojure %}
(defpipe convert-persons-data
  "Pipeline to convert tabular persons data into a different tabular format."
  [data-file]
  (-> (read-dataset data-file)
      (drop-rows 1)
      (make-dataset [:name :sex :age])
      (derive-column :person-uri [:name] base-id)
      (mapc {:age ->integer
             :sex {"f" (s "female")
                   "m" (s "male")}})))
{% endhighlight %}

The first thing to notice about `defpipe` forms is that they are
syntactically identical to Clojure's `defn` form.  This is because
essentially all they do is define a function.  However `defpipe` in
addition to defining a normal Clojure function, also advertises that
function to the plugin and other Grafter services.

For example if you run the command `lein grafter list pipe` the plugin
will search the projects classpath for any clj files with valid
defpipe definitions, and list them.

`defpipe` is necessary as it allows Grafter to distinguish `pipes`
from ordinary functions.

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

### Running a pipe

You should now see the Clojure REPL started in your projects pipeline
namespace.  First lets try running the pipe `convert-persons-data
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

Lets investigate the pipes steps one by one so we can see what's going
on.  First lets try using `read-dataset` to open our example file:

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

## Composing Pipes

A pipe is any function defined with Grafters `defpipe`, that can
convert 0 or more `datasettable` arguments into a Grafter/Incanter
`Dataset` output.

Dataset's are how Grafter represents tabular data, and `datasettables`
are said to be any type which Grafter can coerce into a `Dataset`.  This
includes for example Strings representing file paths or URLs,
`java.io.File` objects, `java.net.URI` and `URL` objects, along with
`InputStream`s and `Reader`s, and various others.  Importantly
`Dataset`s are also `datasettable`.

Typically a `defpipe` definition will as its first action use Grafters
`read-dataset` function to load the supplied object into a Dataset,
before passing this `Dataset` through a series of operations.

Its important to note that pipe's are functions from `datasettable* ->
dataset` and that so long as a pipe meets this contract it can be used
by the Grafter plugin and other Grafter services.

Pipes should always call `read-dataset` on their arguments, to
guarantee that a `defpipe` can be called by the plugin (which will
typically supply it with a file path as a `String`) or by other
pipelines which will pass them an already opened `Dataset`.

This works because calling `read-dataset` on a `Dataset` is guaranteed
to return a `Dataset`; hence `Dataset`s are themselves `datasettable`.

Pipes are typically built as a linear sequence of operations, where
each operation takes an input dataset and some arguments and returns a
new `Dataset` with the changes applied.

Usually we compose this sequence of operations together with Clojure's
thread-first (`->`) macro, which takes the return value of the
previous step and implicitly plugs it into the first argument of the
next function call.

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

So, lets make those headers:

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

Next lets use Grafter's `derive-column` to derive a new URI for each
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

First lets have a play with it at the REPL:

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

So lets try and generate a uri, there is a function defined in the
template for building resource-id URI's already in the
`test-project.prefix` namespace.  Lets try and call it:

<div class="terminal-wrapper">
<div class="terminal-inner">test-project.pipeline=> (base-id "foo")
"http://my-domain.com/id/foo"
</div>
</div>

We can see that this function concatenates its argument to a base
prefix for us (if you want a different prefix you can redifine the
function definition in `test-project.prefix`).

So lets try adding using this with our pipeline so far and
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
