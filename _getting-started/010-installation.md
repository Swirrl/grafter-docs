---
layout: page
title: 1. Creating a Grafter Project
---

**NOTE: This guide covers Grafter 0.3.0**

# Creating a Grafter project

*This is part one of the Grafter Getting Started Guide.*

In this section you'll learn:

- About the dependencies required to use Grafter
- How to create a new Grafter project
- How Grafter projects are structured

## Prerequisites for Grafter

_NOTE_ before you can follow this guide you will need to have
installed
[Java](http://www.oracle.com/technetwork/java/javase/downloads/index.html)
7 or 8 and [Leiningen](http://leiningen.org/), Clojure's build tool.

Grafter, like Clojure is cross platform, and should work on any
platform which can run a Java Virtual Machine.

On Linux you should also be able to install OpenJDK via your package
manager.

## Create a project from our template

Once you've got Leiningen installed, it's really easy to create the
Grafter project template.  You shouldn't need to install anything
else, as leiningen will do everything from now on:

At the command line simply run:

<div class="terminal-wrapper">
  <div class="terminal-inner">$ lein new grafter test-project</div>
</div>

This command downloads our
[grafter-template](https://github.com/Swirrl/grafter-template) from
[clojars](http://clojars.org/) a package repository of open source
Clojure projects, and uses it to generate a very small Grafter
pipeline, an ideal starting point for your transformations.

After running the above command you should see the following output
indicating that the project has been created:

<div class="terminal-wrapper">
  <div class="terminal-inner">$ lein new grafter test-project --snapshot
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

## Grafter Project Layout

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
