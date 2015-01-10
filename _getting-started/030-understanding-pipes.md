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

## What is a pipe?

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

## Running Transformations at the Clojure REPL

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

You'll see that this removes the header row from the data, we can then
choose to set our own headers by issuing a call to `make-dataset`:

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

*If you're curious why the column names argument is in a different
position when we used Clojure's `->` macro, you should read up on
using [thread-first](http://clojuredocs.org/clojure.core/-%3E).*

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
