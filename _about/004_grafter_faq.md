---
layout: page
title: Frequently Asked Questions
---

# Frequently Asked Questions

### How does this relate to Incanter?

[Incanter](http://incanter.org/) is a great Clojure library for data
processing, however its focus is on statistical processing.

We like Incanter so much however, we've built Grafter ontop of their
Datasets to help provide interoperability between ETL and statistical
data tasks.

Our intended focus is more on Extracting, Transforming and Loading
data; rather than the numerical side which Incanter, like R excels at.
In particular we are interested in creating pipelines that convert
tabular data from source files like Spreadsheets and Shapefiles into
Linked Data.

In the future we hope to be able to provide additional tooling in this
area.

### Aren't you just remaking Google/Open Refine?

We love OpenRefine and spent a long time assessing it as an
alternative to Grafter.  Indeed OpenRefine with the RDF plugin is
superficially very close to our vision of end user data transformation
tooling.

Unfortunatley though Open Refine was built primarily to be a desktop
tool for data cleaning, and though people have tried it is not really
suitable for robust ETL workflows.

The biggest barrier to using Open Refine in ETL is that it's primarily
a desktop tool, and the code would require a huge amount of
refactoring to remove its concepts of Workspace, and undo/redo which
are innefficient and unsuitable for our vision of large scale hosted
conversion.

We believe that ETL needs to be reliable, robust and efficient, and
that refine does not provide a suitable framework for building a suite
of complementary tools on top of.

### Aren't you just remaking Pentaho Data Integration (Kettle)

No.  We believe that workflow tools like Pentaho are too complicated
for the majority of users, and too limiting for programmers.

We want to build a tool that allows Spreadsheet and knowledge workers
to be able to build 90% of the transformations they need in an
intuitive manner.  The remaining 10% of transformations might need a
programmer, and that real programming languages are better tools for
this job than graphical workflow environments.
