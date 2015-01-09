---
layout: page
title: Getting Started
---

# Getting Started

Our grafter template installs a minimal but complete Grafter
transformation so you can get started with Grafter right away.

This short guide, will walk you through creating a grafter project
with leiningen and our template project, whilst explaining how to
build, and run grafter pipelines.

Lets get started!

## 1. Install Leiningen

Leiningen is the most popular and well supported build tool for
Clojure development.  It is frequently used to fetch dependencies,
build, execute and configure your Clojure projects.

Clojure and Grafter require the JVM, so if you don't have a recent
version of Java installed you will need to
[install one](http://www.oracle.com/technetwork/java/javase/downloads/index.html)
before setting up Leiningen.

Leiningen will itself be responsible for installing Clojure; so as the
first step you only need to worry about getting Leiningen going, and
you can do that by following the
[installation instructions](http://leiningen.org) on
[leiningen.org](http://leiningen.org/).

## 2. Create a new Grafter project

Now you've got Leiningen, its really easy to create the Grafter
project template.  At the command line simply run:

    $ lein new grafter test-project


This command downloads our
[grafter-template](https://github.com/Swirrl/grafter-template) from
[clojars](http://clojars.org/) a package repository of open source
Clojure projects, and uses it to generate a very small Grafter
pipeline, an ideal starting point for your transformations.

After running the above command you should see the following output
indicating that the project has been created:

<div class="terminal-wrapper">
  <div class="terminal-inner">$ lein new grafter test-project
            ___           __ _
           / __|_ _ __ _ / _| |_ ___ _ _
          | (_ | '_/ _` |  _|  _/ -_) '_|
           \___|_| \__,_|_|  \__\___|_|

      MACHINE TOOLS FOR LINKED DATA MANUFACTURE
                   grafter.org

You can list pipelines defined in this project by running:

  $ lein grafter list

For usage information on the grafter plugin run:

  $ lein help grafter

To run the example pipeline defined in this project run:

  $ cd test-project

  $ lein grafter run test-project.pipeline/import-persons-data ./data/example-data.csv example-output.ttl
</div>
</div>

The output will helpfully suggest some commands that you can run to
check things are working; but before we do that lets look at the
project structure.

## 3. Grafter Project Anatomy

The template will have created a project structure like this:

    .
    ├── README.md
    ├── data
    │   └── example-data.csv
    ├── project.clj
    └── src
        └── test_project
            ├── pipeline.clj
            ├── prefix.clj
            └── transform.clj


The majority of the layout follows that of a standard clojure project
with the source code in `src`, a `project.clj` file and a project
`README.md`.

However if you look in the `project.clj` file you'll see that in
addition to a fairly standard leiningen configuration it specifies a
dependency on the grafter package, and tells the project to use the
grafter leiningen plugin, which provides some additional command line
tools for pipeline developers.

{% highlight clojure %}
(defproject test-project "0.1.0-SNAPSHOT"
  :description "FIXME: this part is for you!"
  :url "http://example.com/FIXME"
  :license {:name "Eclipse Public License"
            :url "http://www.eclipse.org/legal/epl-v10.html"}
  :dependencies [[org.clojure/clojure "1.6.0"]
                 [grafter "0.3.0"]
                 [org.slf4j/slf4j-jdk14 "1.7.5"]]

  :repl-options {:init (set! *print-length* 200)}

  :plugins [[lein-grafter "0.3.0"]])
  {% endhighlight %}

Importantly the template includes some sample data in
`example-data.csv` and two transformations on it, which we'll look at
in the next section.

## 4. Transformation Walkthrough

In this section you'll learn how though Grafter was originally
designed to transform tabular data into linked data (also called RDF),
it also excels at transforming tabular data into tabular data.

In this example you'll see how a grafter can be used to:

- Clean tabular data
- Convert a CSV file into an Excel file or vice versa
- Convert tabular data into Linked Data (RDF)

And to do all of the above concisely, efficiently and reliably.

You'll also learn how to list and run grafter transformations with our
command line tools, how transformations themselves are specified and
how to run them at the REPL.

### Running Transformations from the Commandline

To understand how Grafter works lets first take a look at the example
CSV file (`./data/example-data.csv`) which was installed by the
template:

    name,sex,age
    Alice,f,34
    Bob,m,63

This dataset follows the common pattern in CSV files of specifying the
header on the first row followed by the rows of source data.  Here we
see there are two records, one for Alice, the other Bob listing both
their sex and age.

Next lets take a look at what transformations are defined in this
project, by running the command:

<div class="terminal-wrapper">
  <div class="terminal-inner">$ lein grafter list
test-project.pipeline/convert-persons-data                   data-file                      ;; Pipeline to convert tabular persons data into a different tabular format.
test-project.pipeline/convert-persons-data-to-graph          data-file                      ;; Pipeline to convert the tabular persons data sheet into graph data.</div>
</div>

The command `lein grafter list` is one of the commands provided by the
plugin to list all of the pipelines defined within a project.  It
scans the projects classpath finds all of the pipelines (defined with
`defpipe` or `defgraft`) and displays their name, the arguments they
expect and their documentation string.

### Anatomy of a Transformation

- Grafts and Pipes


### Running Transformations at the Clojure REPL
