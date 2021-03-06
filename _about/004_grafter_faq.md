---
layout: page
title: Frequently Asked Questions
---

# Frequently Asked Questions

### Who made Grafter?

Grafter was built by [Swirrl](http://swirrl.com/) a specialist in
publishing linked open data.  It is being developed as part of the
European FP7 funded [DaPaaS](http://project.dapaas.eu/) and
[Open Cube](http://www.opencube-project.eu/) projects.

### Why Use Clojure?

Because it's the right tool for the job.  ETL is best thought of as
pure functions operating on streams of immutable data.  Clojure
provides many features which in other languages we would need to write
or develop ourselves.  These include:

- Immutable Data Structures
- Being built on the JVM gives us access to excellent libraries and
  performance
- Lazy sequences
- Homoiconicity & the ability to process code as data

Yes, this means you have to learn Clojure to learn Grafter.  If you're
interested in doing this we recommend
[Clojure for the brave and true](http://www.braveclojure.com/) as a
fantastic, free, getting started with the language guide.

### How does this relate to Incanter?

[Incanter](http://incanter.org/) is a great Clojure library for data
processing, however its focus is on statistical processing.

We like Incanter so much however, we've built Grafter on top of their
Datasets to help provide interoperability between ETL and statistical
data tasks.

Our intended focus is more on Extracting, Transforming and Loading
data rather than the numerical side which Incanter - like R - excels at.
In particular we are interested in creating pipelines that convert
tabular data from source files like Spreadsheets and Shapefiles into
linked data.

One of the main differences between Grafter and Incanter is that
Grafter strongly emphasises the need for defining operations as lazy
(stream-like) operations, where as Incanter typically follows an eager
loading approach.

Because ETL is often defining transformations on large datasets, we
believe that where possible the whole Extract, Transform and Load
operation should be done in a single pass, without loading whole
Datasets into memory.

### Aren't you just remaking Google/Open Refine?

We love OpenRefine and spent a long time assessing it as an
alternative to Grafter.  Indeed OpenRefine with the RDF plugin is
superficially very close to our vision of end user data transformation
tooling.

Unfortunately though, Open Refine was built primarily to be a desktop
tool for data cleaning, and although people have tried it is not really
suitable for robust ETL workflows.

The biggest barrier to using Open Refine in ETL is that it's primarily
a desktop tool, and the code would require a huge amount of
refactoring to remove its concepts of Workspace, and undo/redo which
are innefficient and unsuitable for our vision of large scale hosted
conversion.

We believe that ETL needs to be reliable, robust and efficient, and
that OpenRefine does not provide a suitable framework for building a suite
of complementary tools on top of.

Crucially this requires a well defined layered architecture.

### Aren't you just remaking Pentaho Data Integration (Kettle)

No.  We believe that workflow tools like Pentaho are too complicated
for the majority of users, and too limiting for programmers.

We want to build a tool that allows Spreadsheet and knowledge workers
to be able to build 90% of the transformations they need in an
intuitive manner.  The remaining 10% of transformations might need a
programmer, and that real programming languages are better tools for
this job than graphical workflow environments.
